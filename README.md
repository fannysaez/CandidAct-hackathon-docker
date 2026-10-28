# CandidAct — Hackathon

Application web CandidAct : frontend Angular, backend Symfony (API), base de données MariaDB, le tout dockerisé pour un lancement identique chez tous les membres de l'équipe.

**Équipe** : 4 personnes
**Dépôts du projet** :

- `CandidAct-hackathon-docker` — ce dépôt (orchestration Docker)
- `CandidAct-hackathon-frontend` — Angular 19
- `CandidAct-hackathon-backend` — Symfony 7.4

## Sommaire

- [Stack technique](#stack-technique)
- [Structure du projet](#structure-du-projet)
- [Démarrage rapide](#démarrage-rapide)
  - [1. Cloner les 3 dépôts](#1-cloner-les-3-dépôts)
  - [2. Configurer les variables d'environnement](#2-configurer-les-variables-denvironnement)
  - [3. Générer les clés JWT](#3-générer-les-clés-jwt)
  - [4. Lancer le projet](#4-lancer-le-projet)
- [Commandes utiles](#commandes-utiles)
- [Comment ce projet a été construit](#comment-ce-projet-a-été-construit)
- [Sécurité et bonnes pratiques](#sécurité-et-bonnes-pratiques)
- [Documentation complémentaire](#documentation-complémentaire)

## Stack technique

| Composant | Techno | Port local |
|---|---|---|
| Frontend | Angular 19 (CSS, sans SSR) + Bootstrap 5.3.8 | 4200 |
| Backend | Symfony 7.4 (PHP 8.3, Apache), API Platform, Lexik JWT, Nelmio CORS | 8000 |
| Base de données | MariaDB 11 | 3306 |
| Administration DB | phpMyAdmin | 5050 |
| Orchestration | Docker Compose | — |

## Structure du projet

Les 3 dépôts sont indépendants (chacun son propre historique Git), mais clonés imbriqués dans ce dépôt pour plus de confort de travail :

```
CandidAct-hackathon-docker/
├── .git/                           (ce dépôt)
├── .gitignore                      (ignore les 2 dossiers ci-dessous)
├── docker-compose.yml
├── README.md
├── CandidAct-hackathon-backend/    (son propre .git, dépôt séparé)
│   ├── Dockerfile
│   ├── .env                        (NON versionné — à recréer soi-même, voir Sécurité)
│   ├── src/
│   ├── config/
│   │   └── jwt/                    (clés JWT privée/publique, NON versionné — à générer soi-même)
│   └── ...
└── CandidAct-hackathon-frontend/   (son propre .git, dépôt séparé)
    ├── Dockerfile
    ├── src/
    └── ...
```

## Démarrage rapide

### 1. Cloner les 3 dépôts

```bash
git clone https://github.com/fannysaez/CandidAct-hackathon-docker
cd CandidAct-hackathon-docker
git clone https://github.com/fannysaez/CandidAct-hackathon-backend
git clone https://github.com/fannysaez/CandidAct-hackathon-frontend
```

### 2. Configurer les variables d'environnement

Le fichier `.env` du backend est **entièrement ignoré par Git** (il contient les vraies valeurs : base de données, clé d'application, passphrase JWT, CORS). Il n'est donc jamais versionné : chaque membre de l'équipe doit le recréer une fois après son clone, à partir du contenu partagé sur Discord.

```bash
cd CandidAct-hackathon-backend
# Coller le contenu partagé sur Discord dans un nouveau fichier .env
cd ..
```

Variables attendues (exemple) :

```
DATABASE_URL="mysql://root:root@db:3306/candidact?serverVersion=mariadb-11.4.0&charset=utf8mb4"
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=<passphrase partagée par l'équipe>
CORS_ALLOW_ORIGIN='^https?://(localhost|127\.0\.0\.1)(:[0-9]+)?$'
```

### 3. Générer les clés JWT

Les clés privée/publique (`config/jwt/*.pem`) ne sont pas versionnées non plus. Chaque développeur les génère une fois en local, avec la passphrase du `.env` :

```bash
docker compose exec backend php bin/console lexik:jwt:generate-keypair
```

### 4. Lancer le projet

```bash
docker compose up -d --build
```

- Frontend → http://localhost:4200
- Backend → http://localhost:8000
- phpMyAdmin → http://localhost:5050

## Commandes utiles

| Commande | Effet |
|---|---|
| `docker compose up -d --build` | Construit les images (si besoin) et lance tout en arrière-plan |
| `docker compose ps` | Liste les conteneurs et leur statut |
| `docker compose stop` | Arrête les conteneurs sans les supprimer (conserve les données) |
| `docker compose start` | Relance des conteneurs déjà créés |
| `docker compose logs -f backend` | Suit les logs du backend en direct (`frontend` ou `db` aussi) |
| `docker compose exec db mysql -u root -p candidact` | Ouvre une session MariaDB dans le conteneur `db` (mot de passe `root`) |
| `docker compose exec backend php bin/console doctrine:database:create --if-not-exists` | Crée la base applicative si besoin |
| `docker compose exec backend php bin/console doctrine:migrations:migrate` | Applique les migrations Doctrine |
| `docker compose exec backend php bin/console make:entity` | Crée une nouvelle entité Symfony (en local, hors conteneur de préférence) |
| `docker compose exec backend php bin/console lexik:jwt:generate-keypair --overwrite` | Régénère les clés JWT (si erreur d'encodage après changement de passphrase) |

> ⚠️ `docker compose down` supprime les conteneurs et le réseau (mais pas les volumes, sauf avec `-v`). Préférer `docker compose stop` pour une pause simple.

## Comment ce projet a été construit

Résumé des étapes (détail complet dans la documentation PDF, voir plus bas) :

1. Création des 3 dépôts GitHub (frontend, backend, docker)
2. Mise à jour de Node.js via nvm pour satisfaire Angular CLI 19 (≥ 22.22.3)
3. Génération du frontend : `npx @angular/cli@19 new . --directory=. --skip-git` (CSS, sans SSR)
4. Génération du backend : `composer create-project symfony/skeleton .` puis `composer require webapp` (structure complète, Symfony 7.4 à jour)
5. Installation d'API Platform, du bundle Lexik JWT Authentication et de Nelmio CORS pour l'authentification et l'API
6. Conception du modèle de données en Merise (MLD/MPD), traduit en 8 entités Doctrine
7. Écriture des Dockerfile (frontend en mode dev avec `ng serve`, backend PHP 8.3 + Apache, avec `CGIPassAuth On` pour que le jeton JWT soit bien transmis par Apache à PHP)
8. Écriture du `docker-compose.yml` (4 services : backend, frontend, db, phpMyAdmin)
9. Bascule de PostgreSQL vers MariaDB 11 en cours de hackathon (remplacement de pgAdmin par phpMyAdmin)
10. Mise en place du workflow Git d'équipe (branches `main` / `develop` / `EX-USXX/nomUS` par user story, Pull Requests avec review)
11. Build et test de l'ensemble : `docker compose up --build`
12. Sécurisation des identifiants (`.env` entièrement gitignoré) et mise en ligne sur GitHub

## Sécurité et bonnes pratiques

- `.env` contient les vraies valeurs (base de données, `APP_SECRET`, passphrase JWT, CORS) → **jamais versionné**, entièrement ignoré par `.gitignore`, à recréer manuellement par chaque personne qui clone le projet (contenu partagé sur Discord).
- `config/jwt/` (clés privée/publique JWT) → **jamais versionné** non plus, à régénérer en local par chaque développeur avec la passphrase partagée.
- `docker-compose.yml` charge `.env` via `env_file` pour le service `backend`.
- Avant tout `git push`, vérifier que `.env` et `config/jwt/` n'apparaissent jamais dans les fichiers à committer (`git status`).
- Protection de branche recommandée sur `main` (interdiction des force-push) pour éviter d'écraser le travail des autres membres de l'équipe.

## Documentation complémentaire

(Un wiki GitHub pourra être mis en place en complément de ce README, pour centraliser la documentation technique de l'équipe au fil du projet.)
---
