# CandidAct — Hackathon

Application web CandidAct : frontend Angular, backend Symfony (API), base de données PostgreSQL, le tout dockerisé pour un lancement identique chez tous les membres de l'équipe.

**Équipe :** 4 personnes
**Dépôts du projet :**
- [`CandidAct-hackathon-docker`](https://github.com/fannysaez/CandidAct-hackathon-docker) — ce dépôt (orchestration Docker)
- [`CandidAct-hackathon-frontend`](https://github.com/fannysaez/CandidAct-hackathon-frontend) — Angular 19
- [`CandidAct-hackathon-backend`](https://github.com/fannysaez/CandidAct-hackathon-backend) — Symfony 7.4

---

## Sommaire

- [Stack technique](#stack-technique)
- [Structure du projet](#structure-du-projet)
- [Démarrage rapide](#démarrage-rapide)
  - [1. Cloner les 3 dépôts](#1-cloner-les-3-dépôts)
  - [2. Configurer les variables d'environnement](#2-configurer-les-variables-denvironnement)
  - [3. Lancer le projet](#3-lancer-le-projet)
- [Commandes utiles](#commandes-utiles)
- [Comment ce projet a été construit](#comment-ce-projet-a-été-construit)
- [Sécurité et bonnes pratiques](#sécurité-et-bonnes-pratiques)
- [Pièges rencontrés](#pièges-rencontrés)
- [Documentation complémentaire](#documentation-complémentaire)

---

## Stack technique

| Composant   | Techno                        | Port local |
|-------------|--------------------------------|-----------|
| Frontend    | Angular 19 (CSS, sans SSR)     | `4200`    |
| Backend     | Symfony 7.4 (PHP 8.3, Apache)  | `8000`    |
| Base de données | PostgreSQL 16              | `5432`    |
| Orchestration | Docker Compose                | —         |

---

## Structure du projet

Les 3 dépôts sont indépendants (chacun son propre historique Git), mais clonés **imbriqués** dans ce dépôt pour plus de confort de travail :

```
CandidAct-hackathon-docker/
├── .git/                           (ce dépôt)
├── .gitignore                      (ignore les 2 dossiers ci-dessous)
├── docker-compose.yml
├── README.md
├── CandidAct-hackathon-backend/    (son propre .git, dépôt séparé)
│   ├── Dockerfile
│   ├── .env                        (placeholder, versionné)
│   ├── .env.local                  (vraies valeurs, NON versionné — à créer soi-même)
│   ├── src/
│   ├── config/
│   └── ...
└── CandidAct-hackathon-frontend/   (son propre .git, dépôt séparé)
    ├── Dockerfile
    ├── src/
    └── ...
```

---

## Démarrage rapide

### 1. Cloner les 3 dépôts

```bash
git clone https://github.com/fannysaez/CandidAct-hackathon-docker
cd CandidAct-hackathon-docker
git clone https://github.com/fannysaez/CandidAct-hackathon-backend
git clone https://github.com/fannysaez/CandidAct-hackathon-frontend
```

### 2. Configurer les variables d'environnement

Le fichier `.env.local` du backend contient les identifiants réels de la base de données. Il **n'est pas versionné** (volontairement, voir [Sécurité](#sécurité-et-bonnes-pratiques)) : chaque membre de l'équipe doit le créer une fois après son clone.

```bash
cd CandidAct-hackathon-backend
cat > .env.local << 'EOF'
DATABASE_URL="postgresql://app_user:app_password@db:5432/candidact?serverVersion=16&charset=utf8"
EOF
cd ..
```

### 3. Lancer le projet

```bash
docker compose up -d --build
```

- Frontend → http://localhost:4200
- Backend → http://localhost:8000

---

## Commandes utiles

| Commande | Effet |
|---|---|
| `docker compose up -d --build` | Construit les images (si besoin) et lance tout en arrière-plan |
| `docker compose ps` | Liste les conteneurs et leur statut |
| `docker compose stop` | Arrête les conteneurs sans les supprimer (conserve les données) |
| `docker compose start` | Relance des conteneurs déjà créés |
| `docker compose logs -f backend` | Suit les logs du backend en direct (`frontend` ou `db` aussi) |
| `docker compose exec db psql -U app_user -d candidact` | Ouvre une session PostgreSQL dans le conteneur `db` |
| `docker compose exec backend php bin/console doctrine:database:create --if-not-exists` | Crée la base applicative si besoin |
| `docker compose exec backend php bin/console make:entity` | Crée une nouvelle entité Symfony (en local, hors conteneur de préférence) |

> ⚠️ `docker compose down` supprime les conteneurs et le réseau (mais pas les volumes, sauf avec `-v`). Préférer `docker compose stop` pour une pause simple.

---

## Comment ce projet a été construit

Résumé des étapes (détail complet dans la documentation PDF, voir plus bas) :

1. Création des 3 dépôts GitHub (frontend, backend, docker)
2. Mise à jour de Node.js via `nvm` pour satisfaire Angular CLI 19 (≥ 22.22.3)
3. Génération du frontend : `npx @angular/cli@19 new . --directory=. --skip-git` (CSS, sans SSR)
4. Génération du backend : `composer create-project symfony/skeleton .` puis `composer require webapp` (structure complète, Symfony 7.4 à jour — `website-skeleton` étant un package figé/abandonné)
5. Configuration de la base de données PostgreSQL (`.env` / `.env.local`)
6. Écriture des `Dockerfile` (frontend en mode dev avec `ng serve`, backend PHP 8.3 + Apache)
7. Écriture du `docker-compose.yml` (3 services : `backend`, `frontend`, `db`)
8. Build et test de l'ensemble : `docker compose up --build`
9. Sécurisation des identifiants et mise en ligne sur GitHub

---

## Sécurité et bonnes pratiques

- **`.env`** contient uniquement des valeurs **placeholder** → versionné sans risque.
- **`.env.local`** contient les **vraies valeurs** (identifiants PostgreSQL) → jamais versionné, ignoré par `.gitignore`, à recréer manuellement par chaque personne qui clone le projet.
- `docker-compose.yml` charge les deux fichiers via `env_file` (`.env` puis `.env.local`, qui écrase le premier).
- Avant tout `git push`, vérifier que `.env.local` n'apparaît jamais dans les fichiers à committer (`git status`).

---

## Documentation complémentaire

<!-- Une documentation complète au format PDF est disponible (procédure détaillée pas à pas, captures de commandes, explications de chaque choix technique) — à demander à Fanny ou disponible dans le dossier `docs/` du dépôt si ajoutée. -->

*(Un wiki GitHub pourra être mis en place en complément de ce README, pour centraliser la documentation technique de l'équipe au fil du projet.)*