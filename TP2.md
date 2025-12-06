# Mémo TP - Créer et manipuler des conteneurs

## Commandes Podman de base

### Lancer un conteneur interactif
```bash
# Sur le serveur DISTANT
podman container run -ti --name mon_conteneur docker.io/ubuntu:24.04
# -ti = --tty --interactive (mode interactif)
# --name donne un nom au conteneur
```

### Lister les conteneurs
```bash
# Sur le serveur DISTANT
podman container ls # Conteneurs en cours
podman container ls -a # Tous les conteneurs (stoppés aussi)
podman ps # Version courte
podman ps -a # Tous avec version courte
```
### Redémarrer un conteneur spécifique
```bash
# Sur le serveur DISTANT
podman container start <nom_du_conteneur>
# Exemple :
podman container start mon_premier_conteneur
```
### revenir dans un contenur stope en mode inetractif
```bash
# 1 Sur le serveur DISTANT
podman container start <nom ou ID>
podman container exec -it <nom ou ID>
```

### Arrêter et supprimer
```bash
# Sur le serveur DISTANT
podman container stop <nom ou ID> # Arrêter un conteneur
podman container rm <nom ou ID> # Supprimer un conteneur
podman container stop --all # Arrêter tous les conteneurs
podman container prune # Supprimer tous les conteneurs stoppés
```

## Copier fichiers vers/depuis conteneurs

### Environnement → Conteneur
```bash
# Sur le serveur DISTANT
podman container cp fichier.txt mon_conteneur:/chemin/dans/conteneur/
# Exemple : copie vers /root/
podman container cp r2d2.cow mon_premier_conteneur:/root/
```

### Conteneur → Environnement
```bash
# Sur le serveur DISTANT
podman container cp mon_conteneur:/chemin/fichier.txt .
# Exemple : récupérer fichier HTML
podman container cp alpine_asciidoctor:/root/financiers.html .
```

## Tutoriel : Premier conteneur Ubuntu

### 1. Lancer le conteneur
```bash
podman container run --name mon_premier_conteneur --tty --interactive docker.io/ubuntu:24.04
```

### 2. Dans le conteneur (une fois connecté)
```bash
# Vérifier l'OS
cat /etc/os-release
# Installer cowsay
apt update
apt install cowsay
# Tester
/usr/games/cowsay "Bonjour depuis un conteneur !"
# Utiliser fichier personnalisé
/usr/games/cowsay -f /root/r2d2.cow "Beep boop"
# Quitter le conteneur
exit
```

## Exercice : Alpine + Asciidoctor

### 1. Créer conteneur Alpine
```bash
podman container run -ti --name alpine_asciidoctor docker.io/alpine:3.22
```

### 2. Dans le conteneur Alpine
```bash
# Installer Asciidoctor
apk update
apk add asciidoctor
# Vérifier installation
asciidoctor --version
# Convertir fichier Asciidoc
asciidoctor financiers.adoc
# Produit financiers.html
```

## Stratégies déploiement Python

### Stratégie 1 : Ubuntu + Python installé
```bash
# Lancer conteneur Ubuntu
podman container run -ti --name replace_string_container docker.io/ubuntu:24.04
# Dans le conteneur :
apt update
apt install -y python3
# Copier replace_string.py depuis un autre terminal
python3 replace_string.py input.txt output.txt
```

### Stratégie 2 : Image Python directement
```bash
# Lancer conteneur Python (plus rapide)
podman container run -ti --name replace_string_container_v2 docker.io/python:3-slim bash
# Python est déjà installé !
# Copier replace_string.py et tester
python3 replace_string.py input.txt output.txt
```

## Déploiement des différents programmes

### 1. compress_file (Python 3.14 requis)
```bash
# Image Python avec version spécifique
podman container run -ti --name compress_container docker.io/python:3-slim bash
# Dans conteneur, vérifier version
python3 --version
# Copier et tester
python3 compress_file.py fichier.txt
```

### 2. print_colored_string (nécessite colorama)
```bash
podman container run -ti --name color_container docker.io/python:3-slim bash
# Dans conteneur, installer dépendance
pip install colorama
# ou
python3 -m pip install colorama
# Tester
python3 print_colored_string.py "Mon texte"
```

### 3. TicTacToe (Java)
```bash
# Image Java Eclipse Temurin
podman container run -ti --name tictactoe_container docker.io/eclipse-temurin:21-jdk bash
# Dans conteneur, vérifier Java
java --version
javac --version
# Compiler et exécuter
javac TicTacToe.java
java TicTacToe
```

### 4. economic_dispatch (Julia)
```bash
# Image Julia
podman container run -ti --name julia_container docker.io/julia:1.11-trixie bash
# Dans conteneur, vérifier Julia
julia --version
# Installer dépendance DataFrames
julia -e 'import Pkg; Pkg.add("DataFrames")'
# Exécuter
julia economic_dispatch.jl input.txt
```

## Nettoyage serveur de déploiement

### Nettoyage complet Podman
```bash
# Sur le serveur DISTANT
# 1. Stopper tous les conteneurs
podman container stop --all
# 2. Supprimer conteneurs stoppés
podman container prune -f
# 3. Supprimer images non utilisées
podman image prune -a -f
# 4. Supprimer réseaux inutilisés
podman network prune -f
# 5. Supprimer volumes (si utilisés)
podman volume prune -f
# 6. Vérifier l'espace
podman system df
```

### Supprimer fichiers temporaires
```bash
# Sur le serveur DISTANT
rm -rf tp-conteneurs/ # Supprimer répertoire de travail
rm -f *.html *.txt # Supprimer fichiers restants
```

## Astuces importantes

### Points d'entrée personnalisés
```bash
# Spécifier bash comme point d'entrée
podman container run -ti --name mon_conteneur docker.io/python:3-slim bash
# Spécifier sh pour Alpine
podman container run -ti --name mon_conteneur docker.io/alpine:3.22 sh
```

### Vérifier installations
```bash
# Dans les conteneurs, toujours vérifier :
python3 --version # Python
java --version # Java
julia --version # Julia
apt --version # Ubuntu/Debian
apk --version # Alpine
```

### Gestion erreurs courantes
```bash
# "Container already exists"
podman container rm <nom> # Supprimer d'abord
# "Permission denied" dans conteneur
# Êtes-vous root? (normalement oui dans conteneur)
# Image non trouvée
# Vérifier l'étiquette : docker.io/python:3-slim (pas python:3)
```

## Checklist fin TP
1. Tous les conteneurs stoppés
2. Tous les conteneurs supprimés
3. Images nettoyées
4. Fichiers temporaires supprimés
5. Vérifier espace avec `podman system df`
6. Déconnexion SSH propre (`exit`)

**Rappel** : Sur le serveur, vous êtes `root` dans les conteneurs, mais PAS sur le serveur lui-même. Profitez-en pour installer ce dont vous avez besoin dans les conteneurs !
