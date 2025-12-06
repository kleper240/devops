
# Mémo Commandes SSH et Podman

## Connexion SSH au serveur

### 1. Connexion avec mot de passe
```bash
# Sur votre poste LOCAL
ssh EXXXXXX@podman
```
Remplacez `EXXXXXX` par votre ID étudiant en MAJUSCULES.

### 2. Configuration clé SSH (à faire une fois)
```bash
# Sur votre poste LOCAL - À FAIRE UNE FOIS
ssh-keygen -t ecdsa

# Sur votre poste LOCAL
ssh-copy-id -i ~/.ssh/id_ecdsa.pub EXXXXXX@podman
```
Explication : Génère une clé SSH ECDSA si elle n'existe pas, puis copie votre clé SSH publique sur le serveur pour éviter de taper le mot de passe ensuite.

## Transfert de fichiers

### Copier VERS le serveur
```bash
# Sur votre poste LOCAL
scp monfichier EXXXXXX@podman:~                     # Fichier vers home
scp monfichier EXXXXXX@podman:~/mon_repertoire/     # Fichier vers dossier spécifique
scp -r mon_dossier EXXXXXX@podman:~                 # Dossier complet
scp envoi1.txt EXXXXXX@podman:~/tp-decouverte-serveur/   # Exemple vers répertoire spécifique
scp -r envoi2/ EXXXXXX@podman:~/tp-decouverte-serveur/   # Exemple dossier vers répertoire spécifique
```

### Télécharger DEPUIS le serveur
```bash
# Sur votre poste LOCAL
scp EXXXXXX@podman:~/monfichier .                    # Vers répertoire courant
scp EXXXXXX@podman:~/monfichier ./local/             # Vers sous-dossier
scp EXXXXXX@podman:~/fichier.txt .                   # Exemple vers répertoire courant
```

## Édition de fichiers sur le serveur

### Se connecter et éditer
```bash
# 1. D'abord se connecter
ssh EXXXXXX@podman

# 2. Puis sur le serveur DISTANT :
nano fichier.txt
nano hello.txt              # Créer/éditer fichier
```
Contrôles nano : `CTRL+O` (enregistrer), `CTRL+X` (quitter).

### Raccourcis nano
- `CTRL + O` : Enregistrer
- `CTRL + X` : Quitter
- `CTRL + K` : Couper ligne
- `CTRL + U` : Coller
- `CTRL + W` : Rechercher

### Afficher contenu
```bash
# Sur le serveur DISTANT
cat hello.txt               # Affiche contenu fichier
head -n 5 fichier.txt       # Affiche 5 premières lignes
tail -n 5 fichier.txt       # Affiche 5 dernières lignes
```

## Gestion des fichiers sur le serveur

### Navigation et création
```bash
# Sur le serveur DISTANT
pwd                         # Affiche répertoire courant
mkdir tp-decouverte-serveur # Créer répertoire
cd tp-decouverte-serveur    # Se déplacer dedans
ls                          # Lister fichiers
ls -la                      # Lister avec détails
```

## Analyse de l'environnement serveur

### Informations système
```bash
# Sur le serveur DISTANT
cat /etc/os-release         # Distribution et version
hostnamectl                 # Infos détaillées système
uname -a                    # Info noyau
```

### Variables d'environnement
```bash
# Sur le serveur DISTANT
printenv                    # Affiche toutes les variables
export MAVARIABLE="valeur"  # Crée variable
echo $MAVARIABLE            # Affiche variable
env                         # Liste variables (alternative)
```

### Permissions et droits
```bash
# Sur le serveur DISTANT
touch ~/fichier_test        # Créer fichier (devrait fonctionner)
rm ~/fichier_test           # Supprimer fichier
touch /etc/fichier_test     # Échouera (pas de droits)
ls -l fichier.txt           # Voir permissions fichier
```

## Déploiement de programmes Python

### Vérifier Python
```bash
# Sur le serveur DISTANT
python3 --version           # Version Python installée
which python3               # Emplacement Python
```

### Tester dépendances
```bash
# Sur le serveur DISTANT
python3 -c "import sys; print(sys.version)"  # Version détaillée
python3 -c "import zstd"    # Tester module zstd (doit échouer)
python3 -c "import colorama" # Tester module colorama
```

### Exécuter programmes Python
```bash
# Sur le serveur DISTANT
python3 replace_string.py input.txt output.txt
# replace_string.py nécessite Python standard (devrait marcher)
```

### Créer fichiers test
```bash
# Sur le serveur DISTANT
echo "This is a bottle on the table" > test.txt
cat test.txt
```

## Commandes Podman (UNE FOIS CONNECTÉ)

### Vérification
```bash
# Sur le serveur DISTANT (après ssh EXXXXXX@podman)
podman --version
```

### Nettoyage (à faire en fin de session)
```bash
# Sur le serveur DISTANT
podman container stop --all     # Stop tous conteneurs
podman container prune          # Supprime conteneurs stoppés
podman network prune            # Supprime réseaux inutilisés
podman image prune --all        # Supprime images inutilisées
```

### Réinitialisation complète
```bash
# Sur le serveur DISTANT
podman system reset             # Tout remettre à zéro
```

## Commandes utiles supplémentaires

### Recherche et aide
```bash
# Sur le serveur DISTANT
man ls                      # Manuel commande ls
ls --help                   # Aide rapide
find . -name "*.txt"       # Trouver fichiers .txt
grep "motif" fichier.txt   # Rechercher texte
```

### Gestion processus
```bash
# Sur le serveur DISTANT
ps aux                      # Voir processus
top                         # Moniteur système
# CTRL+C                      # Arrêter programme en cours
```

### Archives et compression
```bash
# Sur le serveur DISTANT
tar -czf archive.tar.gz dossier/  # Créer archive
tar -xzf archive.tar.gz           # Extraire archive
gzip fichier.txt                  # Compresser
gunzip fichier.txt.gz             # Décompresser
```

## Problèmes courants et solutions

### "Permission denied"
```bash
# Solution possible :
chmod +x script.py          # Rendre exécutable
```

### "Command not found"
```bash
# Vérifier l'installation :
which podman                # Vérifier si installé
```

### Erreur dépendance Python
```
# Module manquant - NE PAS ESSAYER D'INSTALLER
# Sur serveur, pas de droits pour : pip install ...
```

## Checklist fin de session
```bash
# Sur le serveur DISTANT
# 1. Nettoyer fichiers temporaires
rm -f *.tmp test_*.txt

# 2. Vérifier espace disque
du -sh ~/                   # Taille répertoire home
df -h                       # Espace disque total

# 3. Quitter proprement
exit                        # Déconnexion SSH
```

## Quitter le serveur
```bash
# Sur le serveur DISTANT
exit
```

## Rappels importants

### Ports autorisés
- Uniquement ports 8080 à 9089

### Points clés à retenir
1. SSH : `ssh EXXXXXX@podman` toujours avec ID étudiant
2. Clé SSH : Générer avec `ssh-keygen -t ecdsa` une fois
3. Transferts : `scp` pour fichiers, `ssh` pour connexion
4. Serveur limité : Pas de sudo, pas d'installation logicielle
5. Python limité : Seuls modules standards disponibles
6. Nettoyer : Supprimer fichiers inutiles avant de quitter

Pour TP suivant avec Podman : Gardez en tête que les conteneurs résoudront les problèmes de dépendances rencontrés ici !
