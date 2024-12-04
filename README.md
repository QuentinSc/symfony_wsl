# Environnement de Développement Symfony avec Docker sous WSL2

## Table des matières
1. [Vue d'ensemble](#vue-densemble)
2. [Pourquoi WSL2/Ubuntu ?](#pourquoi-wsl2ubuntu)
3. [Prérequis et installation de l'environnement](#prérequis-et-installation-de-lenvironnement)
4. [Installation du projet](#installation-du-projet)
5. [Architecture technique](#architecture-technique)
6. [Guide de développement](#guide-de-développement)
7. [Dépannage](#dépannage)

## Vue d'ensemble

Cet environnement permet de développer des applications Symfony sous Windows en utilisant :
- WSL2 (Windows Subsystem for Linux) avec Ubuntu
- Docker pour la containerisation
- Nginx comme serveur web
- PHP 8.2-FPM pour l'exécution du code
- MySQL 8.0 comme base de données

## Pourquoi WSL2/Ubuntu ?

L'utilisation de WSL2 avec Ubuntu plutôt que Windows en natif présente plusieurs avantages majeurs :

### Performance
- Les performances I/O (lecture/écriture fichiers) sont nettement meilleures sous WSL2 que via des volumes partagés Windows
- La compilation et l'exécution du code PHP sont plus rapides
- Le système de cache Symfony est plus performant

### Compatibilité
- Environnement Linux natif, identique à la production
- Meilleure compatibilité avec les outils de développement
- Pas de problèmes de fins de ligne (CRLF vs LF)
- Pas de problèmes de casse des fichiers (Windows est insensible à la casse)

### Développement
- Accès à tous les outils CLI Linux natifs
- Meilleure intégration avec Docker (pas de problèmes de volumes)
- Configuration plus proche de l'environnement de production
- Pas besoin de gérer les permissions Windows/Linux

### Ressources
- Utilisation plus efficace des ressources système
- Meilleure isolation des processus
- Pas de surcharge due à la virtualisation complète

## Prérequis et installation de l'environnement

### 1. Windows 11
- Processeur 64 bits avec virtualisation activée dans le BIOS
- Au moins 4GB de RAM recommandés
- Windows 11 build 22000 ou supérieur

### 2. WSL2 et Ubuntu
1. Activer WSL2 dans Windows :
   ```powershell
   # Ouvrir PowerShell en administrateur et exécuter :
   wsl --install
   ```
2. Installer Ubuntu depuis le Microsoft Store
3. Redémarrer votre PC
4. Lancer Ubuntu et créer votre compte utilisateur

### 3. Docker Desktop
1. [Télécharger et installer Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Dans les paramètres de Docker Desktop :
   - Activer l'intégration WSL2
   - Sélectionner Ubuntu dans les distributions WSL2

### 4. IDE (VSCode ou Cursor)
1. Installer [VSCode](https://code.visualstudio.com/) ou [Cursor](https://cursor.sh/)
2. Installer les extensions recommandées :
   - Remote Development (crucial pour WSL)
   - Docker
   - PHP Intelephense

## Installation du projet

### 1. Préparer l'environnement WSL
```bash
# Ouvrir Ubuntu (depuis le menu démarrer ou 'ubuntu' dans PowerShell)
cd ~    # ou cd /home/<votre-user>
```

### 2. Cloner et ouvrir le projet
```bash
# Cloner le dépôt
git clone <votre-depot>
cd <votre-projet>

# Ouvrir dans votre IDE
code .    # pour VSCode
# ou
cursor .  # pour Cursor
```
⚠️ **Important** : Vérifiez que votre IDE indique "WSL: Ubuntu" en bas à gauche

### 3. Installer l'application
```bash
# Créer le dossier app
mkdir app

# Donner les permissions à www-data (utilisateur PHP dans le conteneur)
# Cette commande est cruciale car :
# - PHP s'exécute en tant qu'utilisateur www-data dans le conteneur
# - Composer a besoin d'écrire dans ce dossier pour installer Symfony
# - Symfony a besoin d'écrire dans var/cache, var/log, etc.
# - Sans ces permissions, vous aurez des erreurs d'écriture
sudo chown -R www-data:www-data app

# Démarrer Docker (assurez-vous que Docker Desktop est lancé sur Windows)
docker compose up -d --build

# Installer Symfony
docker exec -it symfony_php composer create-project symfony/skeleton:"7.2.*" .
docker exec -it symfony_php composer require symfony/webapp-pack
# Répondre "n" aux questions sur la configuration Docker
```

### 4. Vérifier l'installation
1. Vérifier l'environnement :
   ```bash
   uname -a    # Doit afficher "Linux"
   docker ps   # Doit montrer vos conteneurs
   ```

2. Accéder à l'application :
   - http://localhost:8080 (page Symfony)
   - http://localhost:8080/lucky/number (exemple de route)

## Architecture technique

### Structure des fichiers
```
projet/
├── .docker/                # Configurations Docker
│   ├── nginx/             # Configuration Nginx
│   │   └── default.conf
│   └── php/              # Configuration PHP
│       └── Dockerfile
├── app/                   # Application Symfony
├── .gitignore
├── README.md
└── docker-compose.yml
```

### Conteneurs Docker
- **symfony_php** : PHP-FPM 8.2 avec extensions
- **symfony_nginx** : Serveur web Nginx
- **symfony_mysql** : Base de données MySQL 8.0

### Base de données
- Host : localhost:3306
- Database : symfony_db
- User : symfony
- Password : symfony

Configuration dans `.env.local` :
```
DATABASE_URL="mysql://symfony:symfony@mysql:3306/symfony_db?serverVersion=8.0"
```

## Guide de développement

### Commandes courantes
```bash
# Gestion des conteneurs
docker compose up -d      # Démarrer
docker compose down      # Arrêter
docker compose logs -f   # Voir les logs

# Commandes Symfony
docker exec -it symfony_php php bin/console cache:clear
docker exec -it symfony_php php bin/console make:controller
docker exec -it symfony_php php bin/console make:entity

# Composer
docker exec -it symfony_php composer require [package]
```

## Dépannage

### Problèmes courants

1. **L'IDE n'est pas en mode WSL**
   - Utiliser Ctrl+Shift+P
   - Chercher "Reopen in Container"
   - Sélectionner WSL: Ubuntu

2. **Erreurs de permission**
   ```bash
   # Cette commande est nécessaire car PHP s'exécute en tant que www-data
   # Si vous avez des erreurs du type :
   # - "Permission denied"
   # - "Failed to create directory"
   # - "Cannot write cache file"
   # - "The directory X is not writable"
   sudo chown -R www-data:www-data app
   ```
   ➜ Pourquoi www-data ?
   - www-data est l'utilisateur standard de PHP dans le conteneur
   - Il a besoin d'accéder et de modifier les fichiers
   - C'est une pratique de sécurité standard dans les conteneurs PHP

3. **Ports déjà utilisés**
   - Vérifier les ports 8080 et 3306
   - Modifier dans docker-compose.yml si nécessaire

4. **Docker ne démarre pas**
   - Vérifier que Docker Desktop est lancé
   - Vérifier l'intégration WSL2 dans les paramètres

### Logs et debugging
```bash
# Logs des conteneurs
docker compose logs -f

# Status Symfony
docker exec -it symfony_php php bin/console about

# Validation base de données
docker exec -it symfony_php php bin/console doctrine:schema:validate
```