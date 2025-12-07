# Mémo TP - Réseaux virtuels et multi-conteneurs

## Création et gestion de réseaux virtuels

### Créer un réseau virtuel
```bash
# Sur le serveur DISTANT
podman network create <nom_reseau>
podman network create r1
```

### Lister les réseaux
```bash
podman network ls
```

### Inspecter un réseau
```bash
podman network inspect <nom_reseau>
```

### Supprimer un réseau
```bash
podman network rm <nom_reseau>
```

## Connecter des conteneurs à un réseau

### Un conteneur sur un réseau
```bash
podman container run -ti --name conteneurA --network r1 docker.io/alpine:3.22
```

### Un conteneur sur plusieurs réseaux
```bash
podman container run -ti --name conteneurB --network r1 --network r2 docker.io/alpine:3.22
```

## Tests réseau avec netcat (nc)

### Installation netcat sur Alpine
```bash
# Dans le conteneur Alpine
apk update && apk add netcat-openbsd
```

### Serveur netcat (écoute)
```bash
nc -l <port>
nc -l 5000
```

### Client netcat (connexion)
```bash
nc <nom_hote> <port>
nc conteneurA 5000
```

## Tutoriel : Mealie + PostgreSQL

### 1. Préparer répertoires
```bash
mkdir -p ~/tp-reseaux/mealie/db-data
mkdir -p ~/tp-reseaux/mealie/app-data
```

### 2. Créer réseau virtuel
```bash
podman network create mealie_network
```

### 3. Lancer PostgreSQL
```bash
podman container run -d \
  --name mealie_db \
  --network mealie_network \
  --mount type=bind,source="$HOME/tp-reseaux/mealie/db-data",target="/var/lib/postgresql/data" \
  -e POSTGRES_USER=mealie \
  -e POSTGRES_PASSWORD=mealie_password \
  -e POSTGRES_DB=mealie_data \
  docker.io/postgres:16
```

### 4. Lancer Mealie
```bash
podman container run -d \
  --name mealie \
  --network mealie_network \
  --publish <VOTRE_PORT>:9000 \
  -e DB_ENGINE=postgres \
  -e POSTGRES_SERVER=mealie_db \
  -e POSTGRES_DB=mealie_data \
  -e POSTGRES_USER=mealie \
  -e POSTGRES_PASSWORD=mealie_password \
  -e POSTGRES_PORT=5432 \
  ghcr.io/mealie-recipes/mealie:v1.0.0-RC1.1
```

## Exercice : Kanboard + MariaDB

### 1. Préparer répertoires
```bash
mkdir -p ~/tp-reseaux/kanboard/db-data
mkdir -p ~/tp-reseaux/kanboard/app-data
```

### 2. Créer réseau virtuel
```bash
podman network create kanboard_network
```

### 3. Lancer MariaDB
```bash
podman container run -d \
  --name kanboard_db \
  --network kanboard_network \
  --mount type=bind,source="$HOME/tp-reseaux/kanboard/db-data",target="/var/lib/mysql" \
  -e MARIADB_USER=kanboard \
  -e MARIADB_PASSWORD=kanboard_password \
  -e MARIADB_DATABASE=kanboard_data \
  -e MARIADB_ROOT_PASSWORD=root_password \
  docker.io/mariadb:11.1
```

### 4. Lancer Kanboard
```bash
podman container run -d \
  --name kanboard_instance \
  --network kanboard_network \
  --publish <VOTRE_PORT>:80 \
  --mount type=bind,source="$HOME/tp-reseaux/kanboard/app-data",target="/var/www/app/data" \
  -e DATABASE_URL="mysql://kanboard:kanboard_password@kanboard_db:3306/kanboard_data" \
  docker.io/kanboard/kanboard:v1.2.32
```

## Exercice avancé : lldap + Kanboard + Mealie

### 1. Plan de déploiement
```
Réseaux à créer :
- lldap_network (pour lldap + apps)
- kanboard_network (Kanboard + MariaDB)
- mealie_network (Mealie + PostgreSQL)
```

### 2. Créer réseaux
```bash
podman network create lldap_network
podman network create kanboard_network
podman network create mealie_network
```

### 3. Déployer lldap
```bash
mkdir -p ~/tp-reseaux/lldap/data
podman container run -d \
  --name lldap_instance \
  --network lldap_network \
  --publish <PORT_LLDAP_WEB>:17170 \
  --mount type=bind,source="$HOME/tp-reseaux/lldap/data",target="/data" \
  -e LLDAP_JWT_SECRET="ma_secrete_jwt" \
  -e LLDAP_KEY_SEED="ma_secrete_key" \
  -e LLDAP_LDAP_BASE_DN="dc=monannuaire" \
  -e LLDAP_LDAP_USER_PASS="motdepasse" \
  docker.io/lldap/lldap:2025-10-14
```

### 4. Kanboard avec LDAP
```bash
# MariaDB comme avant, puis Kanboard avec options LDAP
podman container run -d \
  --name kanboard_instance \
  --network kanboard_network \
  --network lldap_network \
  --publish <PORT_KANBOARD>:80 \
  -e DATABASE_URL="mysql://kanboard:kanboard_password@kanboard_db:3306/kanboard_data" \
  -e LDAP_AUTH=true \
  -e LDAP_BIND_TYPE=user \
  -e LDAP_SERVER="ldap://lldap_instance:3890" \
  -e LDAP_USERNAME="uid=%s,ou=people,dc=monannuaire" \
  -e LDAP_USER_BASE_DN="ou=people,dc=monannuaire" \
  -e LDAP_USER_FILTER="uid=%s" \
  docker.io/kanboard/kanboard:v1.2.32
```

### 5. Mealie avec LDAP
```bash
podman container run -d \
  --name mealie \
  --network mealie_network \
  --network lldap_network \
  --publish <PORT_MEALIE>:9000 \
  -e DB_ENGINE=postgres \
  -e POSTGRES_SERVER=mealie_db \
  -e POSTGRES_DB=mealie_data \
  -e POSTGRES_USER=mealie \
  -e POSTGRES_PASSWORD=mealie_password \
  -e POSTGRES_PORT=5432 \
  -e LDAP_AUTH_ENABLED=true \
  -e LDAP_SERVER_URL="ldap://lldap_instance:3890" \
  -e LDAP_BASE_DN="ou=people,dc=monannuaire" \
  -e LDAP_QUERY_BIND="cn=admin,ou=people,dc=monannuaire" \
  -e LDAP_QUERY_PASSWORD="motdepasse" \
  -e LDAP_ID_ATTRIBUTE="uid" \
  -e LDAP_NAME_ATTRIBUTE="uid" \
  -e LDAP_MAIL_ATTRIBUTE="mail" \
  ghcr.io/mealie-recipes/mealie:v1.0.0-RC1.1
```

## Commandes utiles de diagnostic réseau

### Vérifier conteneurs sur un réseau
```bash
podman container ps --filter network=<nom_reseau>
```

### Tester la connexion depuis un conteneur
```bash
# Dans un conteneur
ping <nom_conteneur>
nc -zv <nom_conteneur> <port>
```

### Voir les IPs attribuées
```bash
podman network inspect <nom_reseau> | grep -A 5 "Containers"
```

## Options importantes pour réseaux

### Alias réseau
```bash
--network-alias <alias> # Nom alternatif pour le conteneur sur le réseau
```

### IP statique (non recommandé)
```bash
--ip <adresse_ip> # Fixer une IP sur le réseau
```

### Network mode
```bash
--network host # Partager l'interface réseau de l'hôte (NON recommandé)
```

## Problèmes courants et solutions

### "Network not found"
```bash
# Vérifier le nom exact
podman network ls
# Créer le réseau si absent
podman network create <nom>
```

### Connexion impossible entre conteneurs
1. Vérifier qu'ils sont sur le même réseau
2. Vérifier les logs des conteneurs
3. Tester avec `ping` depuis un conteneur

### Port déjà utilisé
```bash
# Changer de port
--publish <NOUVEAU_PORT>:<PORT_CONTENEUR>
```

## Nettoyage multi-conteneurs

### Arrêter tous les conteneurs
```bash
podman container stop $(podman container ps -q)
```

### Supprimer tous les conteneurs
```bash
podman container rm $(podman container ps -aq)
```

### Supprimer tous les réseaux
```bash
podman network rm $(podman network ls -q)
```

### Nettoyer répertoires
```bash
rm -rf ~/tp-reseaux/
```

## Bonnes pratiques

### Noms significatifs
```bash
# Bon
--name mealie_db
--name kanboard_app
# Mauvais
--name container1
--name test
```

### Un réseau par service
- Créer un réseau dédié pour chaque groupe de conteneurs liés
- Isoler les services qui n'ont pas besoin de communiquer

### Persistance des données
```bash
# TOUJOURS monter les répertoires de données
--mount type=bind,source="$HOME/mes_donnees",target="/chemin/conteneur"
```

### Ordre de lancement
1. Créer les réseaux
2. Lancer les bases de données
3. Lancer les applications (elles dépendent des bases)

**Rappel** : Les conteneurs sur le même réseau peuvent se parler par leur nom. Les conteneurs sur des réseaux différents ne peuvent pas communiquer directement.
