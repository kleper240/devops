# Mémo TP - Publication de ports réseaux

## Publication de ports avec Podman

### Syntaxe de base
```bash
# Publier port conteneur -> port hôte
--publish <port_hôte>:<port_conteneur>
```

### Exemple nginx
```bash
# Sur le serveur DISTANT
podman container run -d \
  --name serveur_web_1 \
  --rm \
  --publish 8080:80 \
  docker.io/nginx:1.25
```
Important : Utilisez VOTRE port attribué (8080-9089).

## Tutoriel : Serveur web nginx

### 1. Lancer nginx simple
```bash
podman container run -d \
  --name serveur_web_1 \
  --rm \
  --publish <VOTRE_PORT>:80 \
  docker.io/nginx:1.25
```

### 2. Vérifications
```bash
# Voir conteneurs en cours
podman container ps
# Voir publication de ports
podman container port serveur_web_1
# Affiche : 80/tcp -> 0.0.0.0:<VOTRE_PORT>
# Voir logs
podman container logs serveur_web_1
```

### 3. Tester dans navigateur
```
http://podman:<VOTRE_PORT>
```

### 4. Nginx avec montage de répertoire
```bash
# 1. Préparer site web
mkdir -p ~/tp-ports/mon_site_web
cd ~/tp-ports/mon_site_web
# 2. Créer index.html
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Mon Site</title></head>
<body><h1>Bonjour !</h1></body>
</html>
EOF
# 3. Lancer conteneur avec montage
podman container run -d \
  --name serveur_web_2 \
  --rm \
  --publish <VOTRE_PORT>:80 \
  --mount type=bind,source="$PWD",target="/usr/share/nginx/html" \
  docker.io/nginx:1.25
```

### 5. Exécuter commande dans conteneur en cours
```bash
# Ouvrir shell dans conteneur
podman container exec -ti serveur_web_2 bash
# Dans le conteneur :
ls /usr/share/nginx/html
cat /usr/share/nginx/html/index.html
exit
```

## Exercice : Gokapi

### 1. Préparer répertoires de données
```bash
mkdir -p ~/tp-ports/gokapi/data
mkdir -p ~/tp-ports/gokapi/config
```

### 2. Lancer Gokapi
```bash
podman container run -d \
  --name gokapi_instance \
  --publish <VOTRE_PORT>:53842 \
  --mount type=bind,source="$HOME/tp-ports/gokapi/data",target="/app/data" \
  --mount type=bind,source="$HOME/tp-ports/gokapi/config",target="/app/config" \
  docker.io/f0rc3/gokapi:v2.1.0
```

### 3. Configuration initiale
1. Accéder à : `http://podman:<VOTRE_PORT>/setup`
2. **Public Name** : Gokapi de [Votre Nom]
3. **Public Facing URL** : `http://podman:<VOTRE_PORT>`
4. **Username** : admin
5. **Password** : my_password

### 4. Tester persistance
```bash
# 1. Arrêter et supprimer
podman container stop gokapi_instance
podman container rm gokapi_instance
# 2. Relancer avec même commande
# 3. Se reconnecter : les données doivent être conservées
```

## Exercice : Kanboard

### 1. Préparer répertoire données
```bash
mkdir -p ~/tp-ports/kanboard/data
```

### 2. Lancer Kanboard
```bash
podman container run -d \
  --name kanboard_instance \
  --publish <VOTRE_PORT>:80 \
  --mount type=bind,source="$HOME/tp-ports/kanboard/data",target="/var/www/app/data" \
  docker.io/kanboard/kanboard:v1.2.32
```

### 3. Configuration
- URL : `http://podman:<VOTRE_PORT>`
- Login : admin / admin

### 4. Vérifier logs
```bash
podman container logs kanboard_instance
# Doit afficher la clé cryptographique générée
```

## Exercice additionnel : Serveur IRC (InspIRCd)

### 1. Préparer configuration
```bash
mkdir -p ~/tp-ports/irc/config
mkdir -p ~/tp-ports/irc/data
```

### 2. Lancer InspIRCd
```bash
podman container run -d \
  --name irc_server \
  --publish <VOTRE_PORT>:6667 \
  --mount type=bind,source="$HOME/tp-ports/irc/config",target="/inspircd/conf" \
  --mount type=bind,source="$HOME/tp-ports/irc/data",target="/inspircd/data" \
  docker.io/inspircd/inspircd-docker:latest
```

### 3. Se connecter avec client IRC
```bash
# Sur votre poste LOCAL
# Télécharger kirc
./kirc -s podman -p <VOTRE_PORT> -c "#test" -n mon_pseudo
```

## Exercice additionnel : Automatic-Image-Converter

### 1. Préparer répertoires
```bash
mkdir -p ~/tp-ports/aic/inputs
mkdir -p ~/tp-ports/aic/outputs
mkdir -p ~/tp-ports/aic/config
```

### 2. Lancer AIC
```bash
podman container run -d \
  --name aic_instance \
  --publish <VOTRE_PORT>:8080 \
  --mount type=bind,source="$HOME/tp-ports/aic/inputs",target="/inputs" \
  --mount type=bind,source="$HOME/tp-ports/aic/outputs",target="/outputs" \
  --mount type=bind,source="$HOME/tp-ports/aic/config",target="/config" \
  ghcr.io/glenux/automatic-image-converter:latest
```

## Commandes utiles pour gestion ports

### Vérifier ports utilisés
```bash
# Sur le serveur DISTANT
podman container ps --format "table {{.Names}}\t{{.Ports}}"
ss -tlnp | grep <VOTRE_PORT> # Vérifier port système
```

### Dépanner connexion
```bash
# 1. Vérifier conteneur en cours
podman container ps
# 2. Vérifier logs
podman container logs <nom_conteneur>
# 3. Tester depuis serveur
curl http://localhost:<VOTRE_PORT>
# 4. Vérifier pare-feu (normalement ouvert 8080-9089)
```

### Options fréquentes
```bash
-d, --detach # Détacher (arrière-plan)
--rm # Auto-suppression à l'arrêt
-p, --publish # Publication port (raccourci)
-v # Montage (raccourci pour --mount type=bind)
```

## Points critiques

### Ports attribués
- **Utilisez UNIQUEMENT** vos ports 8080-9089 attribués
- Consultez liste Madoc pour connaître votre plage

### Persistance données
```bash
# TOUJOURS monter répertoires pour données persistantes
--mount type=bind,source="$HOME/mes_donnees",target="/chemin/conteneur"
```

### Sécurité
```bash
# NE PAS faire :
--publish 8080:80 # OK si 8080 est votre port
--publish 80:80 # INTERDIT (port système)
--publish 22:22 # INTERDIT (SSH)
```

## Nettoyage fin TP

### Arrêter tous les conteneurs
```bash
podman container stop serveur_web_1 serveur_web_2 gokapi_instance kanboard_instance irc_server aic_instance
```

### Supprimer conteneurs
```bash
podman container rm serveur_web_1 serveur_web_2 gokapi_instance kanboard_instance irc_server aic_instance
```

### Nettoyer répertoires
```bash
# Attention : supprime toutes les données !
rm -rf ~/tp-ports/
```

### Vérifier espace
```bash
podman system df
du -sh ~/tp-ports/
```

## Astuces pratiques

### URL de test
```
http://podman:<PORT> # Applications web
irc://podman:<PORT> # IRC
```

### Ports communs
- 80/443 : HTTP/HTTPS
- 8080 : HTTP alternatif
- 3000 : Développement web
- 5432 : PostgreSQL
- 3306 : MySQL
- 27017 : MongoDB

### Debug connexion
1. Conteneur tourne ? `podman ps`
2. Port publié ? `podman port <nom>`
3. Accessible localement ? `curl localhost:<PORT>`
4. Accessible réseau ? Vérifier pare-feu

**Rappel** : Chaque étudiant a sa plage de ports. Respectez-la pour éviter les conflits !
