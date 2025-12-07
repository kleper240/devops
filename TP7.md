# Mémo TP - Scripter la création d'images de conteneurs

## Authentification registre
```bash
# Sur le serveur DISTANT (Terminal 1)
podman login docker-registry.univ-nantes.fr
# Identifiant: EXXXXXX
# Mot de passe: votre mot de passe Gitlab
```

## Tutoriel : Image cowsay avec Containerfile

### Terminal 1 - Préparer répertoire et Containerfile
```bash
# Sur le serveur DISTANT (Terminal 1)
mkdir -p ~/tp-containerfile/cowsay
cd ~/tp-containerfile/cowsay
# Créer Containerfile v3 (sans CMD)
cat > Containerfile << 'EOF'
FROM docker.io/ubuntu:24.04
RUN apt update
RUN apt install -y cowsay
EOF
```

### Terminal 1 - Construire image v3
```bash
# Sur le serveur DISTANT (Terminal 1)
podman image build --tag tutoriel-cowsay:v3 .
```

### Terminal 1 - Tester image v3
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container run --rm -ti localhost/tutoriel-cowsay:v3
# Dans le conteneur:
/usr/games/cowsay "Bonjour !"
exit
```

### Terminal 1 - Mettre à jour Containerfile avec CMD
```bash
# Sur le serveur DISTANT (Terminal 1)
cat > Containerfile << 'EOF'
FROM docker.io/ubuntu:24.04
RUN apt update
RUN apt install -y cowsay
CMD /usr/games/cowsay $MESSAGE
EOF
```

### Terminal 1 - Construire image v4
```bash
# Sur le serveur DISTANT (Terminal 1)
podman image build --tag tutoriel-cowsay:v4 .
```

### Terminal 1 - Tester image v4 avec variable
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container run --rm \
  --env MESSAGE="Re-bonjour depuis Containerfile !" \
  localhost/tutoriel-cowsay:v4
```

## Exercice : Image Asciidoctor avec Containerfile

### Terminal 2 - Préparer Containerfile
```bash
# Sur le serveur DISTANT (Terminal 2)
mkdir -p ~/tp-containerfile/asciidoctor
cd ~/tp-containerfile/asciidoctor
cat > Containerfile << 'EOF'
FROM docker.io/alpine:3.22
RUN apk update && apk add asciidoctor
CMD ["sh", "-c", "asciidoctor $INPUT_FILE"]
EOF
```

### Terminal 2 - Construire et tester
```bash
# Sur le serveur DISTANT (Terminal 2)
podman image build --tag exercice-asciidoctor:containerfile .
# Tester
mkdir -p test
echo "= Test\n\nContenu" > test/doc.adoc
podman container run --rm \
  --mount type=bind,source="$PWD/test",target=/data \
  --env INPUT_FILE="/data/doc.adoc" \
  localhost/exercice-asciidoctor:containerfile
```

## Tutoriel : Image nginx avec site web

### Terminal 1 - Préparer fichiers
```bash
# Sur le serveur DISTANT (Terminal 1)
mkdir -p ~/tp-containerfile/nginx
cd ~/tp-containerfile/nginx
# Créer index.html
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Site Containerfile</title></head>
<body><h1>Site intégré avec Containerfile</h1></body>
</html>
EOF
# Créer Containerfile
cat > Containerfile << 'EOF'
FROM docker.io/nginx:1.25
RUN rm /usr/share/nginx/html/*
COPY ./*.html /usr/share/nginx/html/index.html
EOF
```

### Terminal 1 - Construire et tester
```bash
# Sur le serveur DISTANT (Terminal 1)
podman image build --tag tutoriel-nginx:containerfile .
# Lancer conteneur
podman container run -d \
  --rm \
  --name nginx_containerfile \
  --publish 8082:80 \
  localhost/tutoriel-nginx:containerfile
# Vérifier
curl http://localhost:8082
```

## Exercice : Web Calculator (Python)

### Terminal 2 - Cloner et préparer
```bash
# Sur le serveur DISTANT (Terminal 2)
cd ~/tp-containerfile
git clone https://gitlab.univ-nantes.fr/gl/developpement_exploitation/resources/web-calculator.git
cd web-calculator
```

### Terminal 2 - Créer Containerfile
```bash
# Sur le serveur DISTANT (Terminal 2)
cat > Containerfile << 'EOF'
FROM docker.io/python:3-slim
WORKDIR /app
COPY . .
RUN pip install --upgrade pip && \
    pip install uvicorn fastapi
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
EOF
```

### Terminal 2 - Construire et tester
```bash
# Sur le serveur DISTANT (Terminal 2)
podman image build --tag webcalculator:containerfile .
# Lancer
podman container run -d \
  --rm \
  --name webcalc_containerfile \
  --publish 8083:8000 \
  localhost/webcalculator:containerfile
# Tester
curl http://localhost:8083
```

### Terminal 2 - Publier sur Gitlab
```bash
# Sur le serveur DISTANT (Terminal 2)
podman image push localhost/webcalculator:containerfile \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/webcalculator:containerfile
```

## Exercice : webdav-embedded-server (Java)

### Terminal 1 - Cloner et préparer
```bash
# Sur le serveur DISTANT (Terminal 1)
cd ~/tp-containerfile
git clone https://gitlab.univ-nantes.fr/gl/developpement_exploitation/resources/webdav-embedded-server.git
cd webdav-embedded-server
```

### Terminal 1 - Créer Containerfile
```bash
# Sur le serveur DISTANT (Terminal 1)
cat > Containerfile << 'EOF'
FROM docker.io/eclipse-temurin:21-jdk
WORKDIR /app
COPY . .
# Compiler avec proxy
RUN gradle -Dhttps.proxyHost=proxy.ensinfo.sciences.univ-nantes.prive \
           -Dhttps.proxyPort=3128 \
           fatJar
# Variables d'environnement pour configuration
ENV USERNAME="admin"
ENV PASSWORD="password"
ENV DATA_DIR="/data"
# Point d'entrée avec variables
CMD ["sh", "-c", "java -jar build/libs/webdav-embedded-server-0.2.1-SNAPSHOT-fatjar.jar --credentials \"${USERNAME}:${PASSWORD}\" --port 8080 ${DATA_DIR}"]
EOF
```

### Terminal 1 - Construire
```bash
# Sur le serveur DISTANT (Terminal 1)
podman image build --tag webdavserver:latest .
```

### Terminal 1 - Tester le déploiement
```bash
# Sur le serveur DISTANT (Terminal 1)
mkdir -p ~/webdav-data
podman container run -d \
  --name webdav_container \
  --rm \
  --publish 8084:8080 \
  --mount type=bind,source="$HOME/webdav-data",target=/data \
  --env USERNAME="monuser" \
  --env PASSWORD="monpass" \
  --env DATA_DIR="/data" \
  localhost/webdavserver:latest
```

### Terminal 1 - Tester connexion WebDAV
1. Sur votre poste LOCAL : Ouvrir "Fichiers"
2. Ctrl+L puis saisir : `dav://podman:8084`
3. Identifiant : `monuser`
4. Mot de passe : `monpass`

### Terminal 1 - Version optimisée (multi-stage)
```bash
cat > Containerfile.optimized << 'EOF'
# Étape 1 : Construction
FROM docker.io/eclipse-temurin:21-jdk AS builder
WORKDIR /app
COPY . .
RUN gradle -Dhttps.proxyHost=proxy.ensinfo.sciences.univ-nantes.prive \
           -Dhttps.proxyPort=3128 \
           fatJar
# Étape 2 : Exécution (plus légère)
FROM docker.io/eclipse-temurin:21-jre
WORKDIR /app
# Copier seulement le JAR depuis l'étape builder
COPY --from=builder /app/build/libs/webdav-embedded-server-0.2.1-SNAPSHOT-fatjar.jar /app/
ENV USERNAME="admin"
ENV PASSWORD="password"
ENV DATA_DIR="/data"
CMD ["sh", "-c", "java -jar webdav-embedded-server-0.2.1-SNAPSHOT-fatjar.jar --credentials \"${USERNAME}:${PASSWORD}\" --port 8080 ${DATA_DIR}"]
EOF
# Construire version optimisée
podman image build -f Containerfile.optimized --tag webdavserver:optimized .
```

## Exercice : WordcloudGenerator (R)

### Terminal 2 - Cloner et préparer
```bash
# Sur le serveur DISTANT (Terminal 2)
cd ~/tp-containerfile
git clone https://gitlab.univ-nantes.fr/gl/developpement_exploitation/resources/WordcloudGenerator.git
cd WordcloudGenerator
```

### Terminal 2 - Créer Containerfile
```bash
# Sur le serveur DISTANT (Terminal 2)
cat > Containerfile << 'EOF'
FROM docker.io/rocker/r-ver:4.3
# Installer dépendances système
RUN apt-get update && \
    apt-get install -y libxml2-dev libcurl4-openssl-dev libssl-dev && \
    rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY . .
# Installer paquets R nécessaires
RUN Rscript -e "install.packages(c('shiny', 'tm', 'wordcloud', 'RColorBrewer', 'XML'))"
EXPOSE 3838
CMD ["R", "-e", "shiny::runApp('/app', host='0.0.0.0', port=3838)"]
EOF
```

### Terminal 2 - Construire (peut être long)
```bash
# Sur le serveur DISTANT (Terminal 2)
podman image build --tag wordcloudgenerator:v1 .
```

### Terminal 2 - Tester
```bash
# Sur le serveur DISTANT (Terminal 2)
# Télécharger un fichier texte test
mkdir -p ~/wordcloud-data
# Copier un fichier .txt ici
podman container run -d \
  --name wordcloud_container \
  --rm \
  --publish 8085:3838 \
  --mount type=bind,source="$HOME/wordcloud-data",target=/data \
  localhost/wordcloudgenerator:v1
```

### Terminal 2 - Accéder à l'application
```
http://podman:8085
```

## Commandes Containerfile utiles

### Instructions courantes
```dockerfile
FROM image:tag # Image de base
RUN command # Exécuter commande pendant build
COPY src dest # Copier fichiers locaux
ADD src dest # Copier + extraction archives
ENV KEY=value # Définir variable d'environnement
WORKDIR /chemin # Changer répertoire de travail
EXPOSE port # Documenter port exposé
CMD ["exec", "args"] # Point d'entrée
ENTRYPOINT ["exec"] # Point d'entrée principal
```

### Variables d'environnement dans CMD
```dockerfile
# Méthode 1 : Shell form avec $VAR
CMD executable $VAR
# Méthode 2 : Exec form avec shell
CMD ["sh", "-c", "executable $VAR"]
```

### Copy patterns
```dockerfile
COPY *.html /dir/ # Tous les .html
COPY file.txt /dir/ # Fichier spécifique
COPY src/ /dir/ # Répertoire entier
```

## Commandes Podman build

### Construire une image
```bash
podman image build --tag nom:tag . # Depuis répertoire courant
podman image build --tag nom:tag --file Chemin/Dockerfile . # Fichier spécifique
```

### Options de build
```bash
--no-cache # Ignorer cache
--build-arg VAR=value # Passer argument de build
--platform linux/amd64 # Spécifier plateforme
```

### Vérifier taille images
```bash
podman image list
podman image inspect nom:tag | grep -A 5 "Size"
```

## Publication sur Gitlab

### Tagger avant push
```bash
podman image tag localhost/nom:tag \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/nom:tag
```

### Push vers Gitlab
```bash
podman image push \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/nom:tag
```

### Pull depuis Gitlab
```bash
podman image pull \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/nom:tag
```

## Nettoyage

### Supprimer images
```bash
podman image rm tutoriel-cowsay:v3 tutoriel-cowsay:v4 \
  exercice-asciidoctor:containerfile tutoriel-nginx:containerfile \
  webcalculator:containerfile webdavserver:latest wordcloudgenerator:v1
```

### Supprimer conteneurs
```bash
podman container stop nginx_containerfile webcalc_containerfile \
  webdav_container wordcloud_container
podman container prune
```

### Nettoyer fichiers
```bash
rm -rf ~/tp-containerfile ~/webdav-data ~/wordcloud-data
```

## Astuces Containerfile

### Optimisation cache
```dockerfile
# Mettre les instructions changeant rarement en haut
FROM ...
RUN apt-get update && apt-get install -y ... # Une seule instruction RUN
COPY package.json . # Copier fichiers de dépendances d'abord
RUN npm install # Installer dépendances
COPY . . # Copier code après
```

### Variables build-time
```dockerfile
ARG VERSION=latest
FROM base:$VERSION
```

### Multi-stage builds
```dockerfile
FROM builder AS build
# Étape de construction
FROM runtime AS final
COPY --from=build /app/binary /app/
# Étape finale plus légère
```

### Santé check
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```

**Important**: Pour Gitlab Université
- Registry: `docker-registry.univ-nantes.fr`
- Projet: `EXXXXXX/developpement_exploitation-tp-images`
- Authentification nécessaire à chaque session SSH
