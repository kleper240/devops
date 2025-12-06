# Mémo TP - Montage de répertoires/fichiers

## Montages avec Podman

### Syntaxe de base pour un montage
```bash
# Sur le serveur DISTANT
podman container run -ti \
  --name mon_conteneur \
  --mount type=bind,source="./chemin/source",target="/chemin/cible" \
  docker.io/ubuntu:24.04
```

### Exemple concret
```bash
# 1. Préparer répertoire source
mkdir -p ~/tp-montage/tutoriel_1
echo "Contenu test" > ~/tp-montage/tutoriel_1/mon_fichier

# 2. Lancer conteneur avec montage
cd ~
podman container run -ti \
  --name conteneur_avec_montage \
  --mount type=bind,source="./tp-montage/tutoriel_1",target="/repertoire_montage" \
  docker.io/ubuntu:24.04
```

## Tutoriel : Montages répertoire et fichier

### Montage de répertoire
```bash
# Source : environnement de déploiement
# Cible : conteneur
--mount type=bind,source="./tp-montage/tutoriel_1",target="/repertoire_montage"
```

### Montage de fichier
```bash
# Source : fichier spécifique
# Cible : chemin dans conteneur
--mount type=bind,source="./tp-montage/tutoriel_1/fichier.txt",target="/root/fichier.txt"
```

### Vérifier dans le conteneur
```bash
# Une fois dans le conteneur
ls /repertoire_montage              # Voir contenu monté
cat /repertoire_montage/mon_fichier # Lire fichier monté
```

## Exercice : Asciidoctor avec montage

### 1. Préparation environnement
```bash
# Sur le serveur DISTANT
mkdir -p ~/tp-montage/asciidoctor
```

### 2. Lancer conteneur avec montage
```bash
podman container run -ti \
  --name alpine_asciidoctor \
  --mount type=bind,source="$HOME/tp-montage/asciidoctor",target="/data" \
  docker.io/alpine:3.22
```

### 3. Dans le conteneur
```bash
# Installer Asciidoctor
apk update
apk add asciidoctor

# Vérifier installation
asciidoctor --version

# Travailler avec fichiers montés
cd /data
asciidoctor financiers.adoc    # Produit financiers.html
```

### 4. Sur environnement de déploiement
```bash
# Copier fichier dans répertoire monté
cp financiers.adoc ~/tp-montage/asciidoctor/

# Vérifier résultat
ls ~/tp-montage/asciidoctor/
# Devrait contenir financiers.adoc ET financiers.html
```

## Exercice : economic_dispatch avec montage

### 1. Préparation
```bash
# Sur le serveur DISTANT
mkdir -p ~/tp-montage/julia
# Copier economic_dispatch.jl et test.txt dans ce répertoire
```

### 2. Lancer conteneur Julia
```bash
podman container run -ti \
  --name julia_container \
  --mount type=bind,source="$HOME/tp-montage/julia",target="/work" \
  docker.io/julia:1.11-trixie bash
```

### 3. Dans le conteneur
```bash
# Installer dépendance
julia -e 'import Pkg; Pkg.add("DataFrames")'

# Exécuter programme
cd /work
julia economic_dispatch.jl test.txt
```

## Exercice : docker-ffmpeg-converter

### 1. Préparation répertoires
```bash
# Sur le serveur DISTANT
mkdir -p ~/tp-montage/docker-ffmpeg-converter/inputs
mkdir -p ~/tp-montage/docker-ffmpeg-converter/outputs
```

### 2. Lancer conteneur avec variables d'environnement
```bash
podman container run -d \
  --name ffmpeg_converter \
  --mount type=bind,source="$HOME/tp-montage/docker-ffmpeg-converter/inputs",target=/inputs \
  --mount type=bind,source="$HOME/tp-montage/docker-ffmpeg-converter/outputs",target=/outputs \
  -e SOURCE_DIRECTORY_PATH=/inputs \
  -e DESTINATION_DIRECTORY_PATH=/outputs \
  -e GLOB_PATTERNS="*.webm" \
  -e SCAN_INTERVAL=5 \
  -e FFMPEG_ARGS="-loglevel error -y -fflags +genpts -i %s -r 24 %s.mp4" \
  -e LOG_FORMAT="text" \
  ghcr.io/kennethwussmann/docker-ffmpeg-converter:1.2.0
```

### 3. Surveiller logs
```bash
# Voir logs en temps réel
podman logs -f ffmpeg_converter

# Voir derniers logs
podman logs ffmpeg_converter
```

### 4. Tester la conversion
```bash
# Sur votre poste LOCAL
# 1. Télécharger vidéo test.webm
# 2. Transférer sur serveur
scp test.webm EXXXXXX@podman:~/tp-montage/docker-ffmpeg-converter/inputs/

# 3. Vérifier logs
podman logs ffmpeg_converter

# 4. Vérifier résultat
ls ~/tp-montage/docker-ffmpeg-converter/outputs/
# Devrait contenir test.webm.mp4
```

### 5. Récupérer résultat
```bash
# Sur votre poste LOCAL
scp EXXXXXX@podman:~/tp-montage/docker-ffmpeg-converter/outputs/test.webm.mp4 .
```

## Variables d'environnement avec Podman

### Syntaxe pour variables
```bash
# Option -e pour chaque variable
podman container run -ti \
  -e MA_VARIABLE="valeur" \
  -e AUTRE_VARIABLE="autre" \
  docker.io/ubuntu:24.04

# Dans le conteneur, vérifier
echo $MA_VARIABLE
```

### Variables pour docker-ffmpeg-converter
```bash
-e SOURCE_DIRECTORY_PATH=/inputs
-e DESTINATION_DIRECTORY_PATH=/outputs
-e GLOB_PATTERNS="*.webm"
-e SCAN_INTERVAL=5
-e FFMPEG_ARGS="-loglevel error -y -fflags +genpts -i %s -r 24 %s.mp4"
-e LOG_FORMAT="text"
```

## Commandes utiles pour TP

### Vérifier montages existants
```bash
# Voir configuration conteneur
podman container inspect <nom_conteneur> | grep -A 10 Mounts
```

### Arrêter conteneur en arrière-plan
```bash
# -d pour détaché (arrière-plan)
podman container run -d --name mon_conteneur ...

# Voir conteneurs en cours
podman ps

# Stopper conteneur
podman container stop <nom>
```

### Gérer logs
```bash
# Suivre logs en temps réel
podman logs -f <nom_conteneur>

# Voir derniers N lignes
podman logs --tail=50 <nom_conteneur>
```

## Nettoyage après TP

### Supprimer conteneurs de ce TP
```bash
# Arrêter tous les conteneurs
podman container stop conteneur_avec_montage conteneur_avec_montage_2 alpine_asciidoctor julia_container ffmpeg_converter

# Supprimer
podman container rm conteneur_avec_montage conteneur_avec_montage_2 alpine_asciidoctor julia_container ffmpeg_converter
```

### Supprimer répertoires de travail
```bash
# Attention : supprime aussi les fichiers générés !
rm -rf ~/tp-montage/
```

### Vérifier espace
```bash
podman system df
du -sh ~/tp-montage/
```

## Points importants

### Chemins relatifs vs absolus
```bash
# Relatif (depuis ~) - ATTENTION : fonctionne seulement si on est dans ~
--mount type=bind,source="./tp-montage",target="/data"

# Absolu - PLUS SÛR
--mount type=bind,source="$HOME/tp-montage",target="/data"
```

### Permissions fichiers montés
- Les fichiers créés dans le conteneur auront les permissions du conteneur
- Mais sur l'hôte, ils auront vos permissions utilisateur
- Attention aux fichiers créés en root dans le conteneur : sudo peut être nécessaire pour les supprimer sur l'hôte

### Avantages montages vs podman cp
- Plus rapide (pas de copie)
- Synchronisation automatique
- Idéal pour développement
- Meilleur pour fichiers volumineux
