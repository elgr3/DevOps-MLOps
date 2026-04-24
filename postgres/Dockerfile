FROM postgres:17.2-alpine

# Les variables POSTGRES_DB / POSTGRES_USER / POSTGRES_PASSWORD
# sont injectées au runtime via le fichier .env (voir docker-compose.yml).
# Ne JAMAIS les hardcoder   ici : l'image Dockerfile est publiée sur DockerHub.

COPY *.sql /docker-entrypoint-initdb.d/