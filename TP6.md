# Mémo TP - Créer manuellement des images de conteneurs

## Authentification registre Université

### Se connecter au registre
```bash
# Sur le serveur DISTANT (Terminal 1)
podman login docker-registry.univ-nantes.fr
# Identifiant: EXXXXXX (votre ID étudiant)
# Mot de passe: votre mot de passe Gitlab
```
Important: À refaire à chaque nouvelle session SSH.

## Tutoriel : Image Ubuntu avec cowsay

### Terminal 1 - Créer conteneur template
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container run -ti --name cowsay_template docker.io/ubuntu:24.04
```

### Dans le conteneur - Installer cowsay
```bash
# Dans le conteneur (Terminal 1)
apt update
apt install cowsay
exit
```

### Terminal 1 - Créer image v1 (sans CMD)
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container commit cowsay_template tutoriel-cowsay:v1
```

### Terminal 1 - Tester image v1
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container run --rm -ti localhost/tutoriel-cowsay:v1
# Dans le conteneur :
/usr/games/cowsay "Bonjour !"
exit
```

### Terminal 1 - Créer image v2 (avec CMD)
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container commit cowsay_template \
  --change 'CMD /usr/games/cowsay $MESSAGE' \
  tutoriel-cowsay:v2
```

### Terminal 1 - Tester image v2 avec variable
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container run --rm \
  --env MESSAGE="Re-bonjour depuis notre première image !" \
  localhost/tutoriel-cowsay:v2
```

### Terminal 1 - Publier images sur Gitlab
```bash
# Sur le serveur DISTANT (Terminal 1)
# Version 1
podman image push localhost/tutoriel-cowsay:v1 \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/tutoriel-cowsay:v1

# Version 2
podman image push localhost/tutoriel-cowsay:v2 \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/tutoriel-cowsay:v2
```

## Exercice : Image Alpine avec Asciidoctor

### Terminal 2 - Préparer conteneur template
```bash
# Sur le serveur DISTANT (Terminal 2)
podman container run -ti --name asciidoctor_template docker.io/alpine:3.22
```

### Dans le conteneur - Installer Asciidoctor
```bash
# Dans le conteneur (Terminal 2)
apk update
apk add asciidoctor
exit
```

### Terminal 2 - Créer image avec point d'entrée
```bash
# Sur le serveur DISTANT (Terminal 2)
podman container commit asciidoctor_template \
  --change 'CMD ["asciidoctor", "$INPUT_FILE"]' \
  exercice-asciidoctor:1.0
```

### Terminal 2 - Tester l'image
```bash
# 1. Préparer fichier test
mkdir -p ~/tp-images/asciidoc
echo "= Titre\n\nContenu test" > ~/tp-images/asciidoc/test.adoc

# 2. Lancer conteneur avec montage
podman container run --rm \
  --mount type=bind,source="$HOME/tp-images/asciidoc",target="/data" \
  --env INPUT_FILE="/data/test.adoc" \
  localhost/exercice-asciidoctor:1.0
```

### Terminal 2 - Publier image
```bash
# Sur le serveur DISTANT (Terminal 2)
podman image push localhost/exercice-asciidoctor:1.0 \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/exercice-asciidoctor:1.0
```

## Tutoriel : Image nginx avec site web intégré

### Terminal 1 - Créer conteneur template
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container run -ti --name nginx_template docker.io/nginx:1.25 bash
```

### Dans le conteneur - Préparer répertoire web
```bash
# Dans le conteneur (Terminal 1)
rm /usr/share/nginx/html/*
exit
```

### Terminal 2 - Copier fichier HTML
```bash
# Sur le serveur DISTANT (Terminal 2)
# Préparer fichier HTML
mkdir -p ~/tp-images/nginx
cat > ~/tp-images/nginx/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Mon Site</title></head>
<body><h1>Site intégré dans l'image!</h1></body>
</html>
EOF

# Copier dans conteneur
podman container cp ~/tp-images/nginx/index.html nginx_template:/usr/share/nginx/html/
```

### Terminal 1 - Créer image nginx
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container commit nginx_template \
  --change 'CMD nginx -g "daemon off;"' \
  tutoriel-nginx:1
```

### Terminal 1 - Tester l'image
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container run -d \
  --name nouveau_serveur_web \
  --rm \
  --publish 8080:80 \
  localhost/tutoriel-nginx:1

# Vérifier
curl http://localhost:8080
```

### Terminal 1 - Publier image
```bash
# Sur le serveur DISTANT (Terminal 1)
podman image push localhost/tutoriel-nginx:1 \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/tutoriel-nginx:1
```

## Exercice : Image economic_dispatch (Julia)

### Terminal 2 - Préparer conteneur template
```bash
# Sur le serveur DISTANT (Terminal 2)
podman container run -ti --name julia_template docker.io/julia:1.11-trixie bash
```

### Dans le conteneur - Installer dépendances
```bash
# Dans le conteneur (Terminal 2)
# Créer répertoire pour code
mkdir /app
cd /app

# Installer DataFrames
julia -e 'import Pkg; Pkg.add("DataFrames")'
```

### Terminal 3 - Copier code source
```bash
# Sur le serveur DISTANT (Terminal 3)
# Cloner ou créer fichier economic_dispatch.jl
mkdir -p ~/tp-images/julia
# Copier le fichier economic_dispatch.jl ici

# Copier dans conteneur
podman container cp ~/tp-images/julia/economic_dispatch.jl julia_template:/app/
```

### Terminal 2 - Créer image
```bash
# Dans le conteneur (Terminal 2)
exit

# Sur le serveur DISTANT (Terminal 2)
podman container commit julia_template \
  --change 'WORKDIR /app' \
  --change 'CMD ["julia", "economic_dispatch.jl", "$INPUT_FILE"]' \
  exercice-economic-dispatch:a
```

### Terminal 2 - Tester l'image
```bash
# Sur le serveur DISTANT (Terminal 2)
# Préparer fichier d'entrée
cat > ~/tp-images/julia/input.txt << 'EOF'
# Contenu exemple du README
EOF

# Lancer conteneur
podman container run --rm \
  --mount type=bind,source="$HOME/tp-images/julia",target="/data" \
  --env INPUT_FILE="/data/input.txt" \
  localhost/exercice-economic-dispatch:a
```

### Terminal 2 - Publier image
```bash
# Sur le serveur DISTANT (Terminal 2)
podman image push localhost/exercice-economic-dispatch:a \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/exercice-economic-dispatch:a
```

## Exercice : Image Web Calculator (Python)

### Terminal 1 - Préparer conteneur template
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container run -ti --name webcalc_template docker.io/python:3-slim bash
```

### Dans le conteneur - Installer dépendances
```bash
# Dans le conteneur (Terminal 1)
# Mettre à jour pip
python3 -m pip install --upgrade pip

# Installer dépendances de base
pip install uvicorn fastapi
```

### Terminal 2 - Copier code source
```bash
# Sur le serveur DISTANT (Terminal 2)
mkdir -p ~/tp-images/webcalc
# Copier tous les fichiers du projet web-calculator ici

# Copier dans conteneur
podman container cp ~/tp-images/webcalc/ webcalc_template:/app/
```

### Terminal 1 - Finaliser installation
```bash
# Dans le conteneur (Terminal 1)
cd /app
# Installer autres dépendances si nécessaire
# pip install -r requirements.txt (si existe)
exit
```

### Terminal 1 - Créer image
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container commit webcalc_template \
  --change 'WORKDIR /app' \
  --change 'CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]' \
  webcalculator:dev
```

### Terminal 1 - Tester l'image
```bash
# Sur le serveur DISTANT (Terminal 1)
podman container run -d \
  --name webcalc_test \
  --rm \
  --publish 8081:8000 \
  localhost/webcalculator:dev

# Vérifier
curl http://localhost:8081
```

### Terminal 1 - Publier image
```bash
# Sur le serveur DISTANT (Terminal 1)
podman image push localhost/webcalculator:dev \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/webcalculator:dev
```

## Commandes utiles pour gestion images

### Lister images locales
```bash
# Sur le serveur DISTANT
podman image list
podman image ls
```

### Voir détails d'une image
```bash
podman image inspect <nom_image>:<tag>
```

### Supprimer images locales
```bash
podman image rm <nom_image>:<tag>
podman image prune  # Supprimer images non utilisées
```

### Tagger une image (changer nom/tag)
```bash
podman image tag localhost/tutoriel-cowsay:v1 \
  docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/nouveau-nom:v2
```

### Pull image depuis registre
```bash
podman image pull docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/tutoriel-cowsay:v1
```

## Structure recommandée pour vos projets

### Organisation fichiers
```
~/tp-images/
├── asciidoc/          # Fichiers pour Asciidoctor
├── nginx/            # Fichiers HTML pour nginx
├── julia/            # Fichiers Julia
└── webcalc/          # Fichiers Python
```

### Points d'entrée courants (CMD)
```bash
# Pour applications web
--change 'CMD ["nginx", "-g", "daemon off;"]'
--change 'CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]'

# Pour scripts avec variables
--change 'CMD ["sh", "-c", "asciidoctor $INPUT_FILE"]'

# Pour interpréteurs
--change 'CMD ["julia", "script.jl", "$ARG"]'
```

## Nettoyage fin TP

### Supprimer conteneurs templates
```bash
# Sur le serveur DISTANT
podman container rm cowsay_template asciidoctor_template nginx_template julia_template webcalc_template
```

### Supprimer images locales
```bash
podman image rm tutoriel-cowsay:v1 tutoriel-cowsay:v2 exercice-asciidoctor:1.0 tutoriel-nginx:1 exercice-economic-dispatch:a webcalculator:dev
```

### Nettoyer répertoires
```bash
rm -rf ~/tp-images/
```

### Vérifier espace
```bash
podman system df
```

## Bonnes pratiques

### Noms et tags
- Utilisez des noms explicites : `monapp-web`, `monapp-api`
- Versionnez avec tags : `v1.0`, `v1.1`, `latest`
- Pour développement : `dev`, `test`

### Points d'entrée
- Toujours spécifier un `CMD` explicite
- Utiliser la syntaxe tableau : `["executable", "arg1", "arg2"]`
- Pour variables : `["sh", "-c", "commande $VAR"]`

### Publication
- Tagger avant push : `podman tag local/image registry/chemin/image:tag`
- Vérifier sur Gitlab : Déploiement → Registre de conteneur
- Authentification nécessaire à chaque session

**Important pour Gitlab** : 
- Projet : `developpement_exploitation-tp-images`
- Identifiant : `EXXXXXX`
- URL registry : `docker-registry.univ-nantes.fr/EXXXXXX/developpement_exploitation-tp-images/`
