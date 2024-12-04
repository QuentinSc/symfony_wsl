# Symfony Docker Development Environment

## Prérequis
- Windows 11
- Docker Desktop
- WSL2 (Windows Subsystem for Linux 2)
- Git

## Installation

1. Cloner ce dépôt :
```powershell
git clone <votre-depot>
cd <votre-projet>
```

2. Copier le fichier d'environnement :
```powershell
cp app/.env app/.env.local
```
Puis modifiez les variables dans `.env.local` selon vos besoins.

3. Lancer les conteneurs Docker :
```powershell
docker compose up -d
```

4. Installer les dépendances Symfony :
```powershell
docker exec -it symfony_php composer create-project symfony/skeleton .
```

5. Corriger les permissions si nécessaire :
```powershell
docker exec -it symfony_php chown -R www-data:www-data .
```

6. Vérifier l'installation :
- Application : http://localhost:8080
- Base de données : localhost:3306
  - Database : symfony_db
  - Username : symfony
  - Password : symfony

## Structure du projet
```
symfony_docker/
├── .docker/               # Configurations Docker (PHP, Nginx)
│   ├── nginx/            # Configuration Nginx
│   └── php/             # Configuration PHP-FPM
├── app/                   # Application Symfony
├── docs/                  # Documentation du projet
├── scripts/              # Scripts utilitaires
├── .gitignore           # Fichiers à ignorer par Git
└── docker-compose.yml    # Configuration Docker Compose
```

## Commandes utiles

- Démarrer les conteneurs : `docker compose up -d`
- Arrêter les conteneurs : `docker compose down`
- Logs des conteneurs : `docker compose logs -f`
- Accéder au conteneur PHP : `docker exec -it symfony_php bash`
- Installer des dépendances : `docker exec -it symfony_php composer require [package]`
- Créer une entité : `docker exec -it symfony_php php bin/console make:entity`
- Créer une migration : `docker exec -it symfony_php php bin/console make:migration`
- Exécuter les migrations : `docker exec -it symfony_php php bin/console doctrine:migrations:migrate`

## Structure des conteneurs
- PHP 8.2 (FPM)
- Nginx (dernière version stable)
- MySQL 8.0

## Notes importantes
- Assurez-vous que Docker Desktop est en cours d'exécution avant de lancer les commandes
- Vérifiez que WSL2 est correctement configuré dans Docker Desktop
- Utilisez PowerShell ou Windows Terminal pour une meilleure expérience
- Les ports utilisés (8080, 3306) doivent être disponibles sur votre machine

## Résolution des problèmes courants

1. Si vous avez des problèmes de permissions :
```powershell
docker exec -it symfony_php chown -R www-data:www-data .
```

2. Si les conteneurs ne démarrent pas, vérifiez :
- Que Docker Desktop est bien lancé
- Qu'aucun autre service n'utilise les ports 8080 et 3306
- Les logs avec `docker compose logs -f`

## Contribution
Les contributions sont les bienvenues ! N'hésitez pas à ouvrir une issue ou une pull request.