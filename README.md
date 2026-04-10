# 🐳 Docker Compose — Stack Web 3 Tiers

Stack conteneurisée composée d'une base de données PostgreSQL, d'un backend Spring Boot, et d'un reverse proxy Apache (httpd).

---

## Architecture

```
Client → httpd (port 80) → backend (Spring Boot) → database (PostgreSQL)
```

Les trois services communiquent via un réseau Docker interne (`app-network`). Seul le port 80 est exposé à l'extérieur.

---

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) ≥ 24
- [Docker Compose](https://docs.docker.com/compose/) ≥ 2.20

---

## Structure du projet

```
.
├── docker-compose.yml
├── .env                  # Variables d'environnement (à créer)
├── postgres/             # Dockerfile + scripts init PostgreSQL
├── simpleapi/simpleapi/  # Dockerfile du backend Spring Boot
└── httpd/                # Dockerfile + config Apache
```

---

## Configuration

Crée un fichier `.env` à la racine du projet :

```env
POSTGRES_USER=usr
POSTGRES_PASSWORD=motdepasse
POSTGRES_DB=db
```

> ⚠️ Ne jamais committer ce fichier. Ajoute `.env` à ton `.gitignore`.

---

## Lancer la stack

```bash
# Démarrer tous les services
docker compose up -d

# Voir les logs en temps réel
docker compose logs -f

# Arrêter la stack
docker compose down

# Arrêter et supprimer les volumes (⚠️ supprime les données)
docker compose down -v
```

---

## Services

### `database` — PostgreSQL

| Propriété | Valeur |
|-----------|--------|
| Image | Build custom (`./postgres`) |
| Réseau | `app-network` (interne) |
| Volume | `postgres-data` (persistant) |
| Healthcheck | `pg_isready` toutes les 10s |
| Redémarrage | `unless-stopped` |

La base de données est considérée "prête" uniquement lorsque `pg_isready` répond avec succès. Les autres services attendent ce signal avant de démarrer.

---

### `backend` — Spring Boot

| Propriété | Valeur |
|-----------|--------|
| Image | Build custom (`./simpleapi/simpleapi`) |
| Réseau | `app-network` (interne) |
| Dépend de | `database` (condition : `service_healthy`) |
| Healthcheck | `GET /health` sur le port 8080 |
| Redémarrage | `unless-stopped` |

Le backend ne démarre qu'une fois la base de données saine. Un healthcheck vérifie ensuite que l'API répond avant que httpd ne soit lancé.

> 💡 **À adapter** : le chemin `/health` et le port `8080` doivent correspondre à l'endpoint de santé réel de ton application Spring Boot.

---

### `httpd` — Apache Reverse Proxy

| Propriété | Valeur |
|-----------|--------|
| Image | Build custom (`./httpd`) |
| Port exposé | `80:80` |
| Réseau | `app-network` |
| Dépend de | `backend` (condition : `service_healthy`) |
| Redémarrage | `unless-stopped` |
| Logs | JSON, max 10 Mo × 3 fichiers |

Point d'entrée unique de la stack. Toutes les requêtes externes passent par ce conteneur, qui les redirige vers le backend.

---

## Réseau

Un réseau bridge `app-network` est partagé entre les trois services. Les conteneurs s'y joignent par leur nom (ex: `http://backend:8080`).

---

## Volume

`postgres-data` est un volume Docker nommé. Les données PostgreSQL y sont persistées même après un `docker compose down`.

Pour inspecter le volume :

```bash
docker volume inspect postgres-data
```

---

## Limites de ressources

Chaque service est contraint en mémoire et CPU pour éviter la saturation de la machine hôte :

| Service  | Mémoire max | CPU max |
|----------|-------------|---------|
| database | 512 Mo      | 0.5     |
| backend  | 256 Mo      | 0.5     |
| httpd    | 128 Mo      | 0.25    |

---

## Healthchecks & ordre de démarrage

```
database ──(healthy)──▶ backend ──(healthy)──▶ httpd
```

Chaque service attend que le précédent soit déclaré sain avant de démarrer. Cela évite les erreurs de connexion au boot.

---

## Dépannage

```bash
# Vérifier l'état des conteneurs
docker compose ps

# Inspecter le healthcheck d'un service
docker inspect postgres-container | grep -A 10 Health

# Relancer uniquement un service
docker compose restart backend

# Accéder au shell d'un conteneur
docker exec -it backend sh
```