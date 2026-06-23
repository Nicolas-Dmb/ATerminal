# aterminal

Outil d'analyse de code : il parse un dépôt, en extrait les modules et symboles,
puis expose et visualise le **graphe de modules** d'un projet.

## Architecture

Trois services orchestrés par Docker Compose :

| Service | Stack | Rôle | Port |
|---------|-------|------|------|
| **engine** | Rust | Analyse le code (tree-sitter), extrait modules & symboles vers PostgreSQL. Déclenché via `LISTEN/NOTIFY` Postgres. | aucun (écoute la base) |
| **api** | Python / FastAPI | Expose les données d'analyse (ex. `/graph/{run_id}`). | 8000 |
| **frontend** | React / Vite | Affiche le graphe de modules. | 3000 |

> Les conteneurs tournent en utilisateur **non-root**. L'engine est compilé en
> multi-stage (image finale `debian-slim`, sans la toolchain Rust).

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) + Docker Compose
- Une base **PostgreSQL** joignable depuis les conteneurs (non fournie par ce
  `docker-compose.yml`)
- Les fichiers d'environnement de chaque service (voir ci-dessous)

## Configuration

Crée un fichier `.env` dans chaque service.

**`engine/.env`**
```env
DATABASE_URL=postgres://user:password@host:5432/dbname
```

**`api/.env`**
```env
DB_USER=user
DB_PASSWORD=password
DB_NAME=dbname
DB_HOST=host
DB_PORT=5432
CORS_ORIGINS=["http://localhost:3000"]
# Optionnels
ENV=dev
LOG_LEVEL=INFO
```

**`frontend/.env`**
```env
# Optionnel — défaut: http://localhost:8000
VITE_API_URL=http://localhost:8000
```

## Lancer

```bash
docker compose build
docker compose up -d
```

## Arrêter

```bash
docker compose down       # arrête et supprime les conteneurs
docker compose down -v    # idem + supprime les volumes
```

## Logs

```bash
docker compose logs <service>        # service = engine | api | frontend
docker compose logs engine -f        # suivre en direct (Ctrl+C pour quitter)
docker compose logs engine --tail 20 # les 20 dernières lignes
```

## Développement local

Chaque service a son propre README avec les commandes de dev natives (sans Docker) :

- [`engine/README.md`](engine/README.md) — lancer l'engine, ajouter une migration, ajouter un adaptateur de langage
- [`api/README.md`](api/README.md) — serveur de dev FastAPI
- [`frontend/README.md`](frontend/README.md) — dev Vite/React
