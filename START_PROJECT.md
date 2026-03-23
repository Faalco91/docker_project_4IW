# START_PROJECT

## 1. Étapes pour construire et exécuter les conteneurs

À la racine du projet.

**Développement**

```bash
docker compose up --build
```

**Production**

```bash
cp .env.production.example .env.production
```

ajouter `JWT_SECRET` dans `.env.production`.

```bash
docker compose -f docker-compose.prod.yml --env-file .env.production up --build -d
```

## 2. Configurations spécifiques à chaque environnement (dev / prod)

**Développement**

- Compose : `docker-compose.yml`
- Dockerfiles : cible `development`
- Code : volumes pour le hot-reload
- `JWT_SECRET` : dans `docker-compose.yml`
- Frontend : Vite sur le port 8080
- APIs sur l’hôte : 3001 (auth), 3000 (produits), 3002 (commandes)
- MongoDB : volumes `mongo_*_data`

**Production**

- Compose : `docker-compose.prod.yml` + `--env-file .env.production`
- Dockerfiles : cible `production`
- Code : uniquement dans l’image (pas de volumes du code)
- `JWT_SECRET` : dans `.env.production` (non versionné)
- Frontend : Nginx, fichiers statiques, port 8080
- APIs sur l’hôte : 8080 via Nginx
- MongoDB : volumes `mongo_*_prod_data`

## 3. Commandes pour tester les services

**Développement**

```bash
curl -s http://localhost:3001/api/health
curl -s http://localhost:3000/api/health
curl -s http://localhost:3002/api/health
curl -s http://localhost:3000/api/products
```

**Production**

```bash
curl -s http://localhost:8080/api/products
```

## 4. Bonnes pratiques appliquées

- Réduire la taille des images : base `node:20-bookworm-slim` pour les services Node ; `npm install --omit=dev` dans les stages production** des Dockerfiles backend (les micro services auth, product, order) ; frontend prod en multi-stage.
- Ne pas exécuter les APIs Node en root : `chown -R node:node /app` puis `USER node` dans ces trois Dockerfiles. Le frontend prod repose sur l’image Nginx officielle.
- Gérer les secrets hors dépôt : `.env` et `.env.production` dans `.gitignore` ; en prod, `JWT_SECRET` via `.env.production` et `env_file` dans `docker-compose.prod.yml` ; `.env.production.example` versionné comme modèle.
- Exclure les fichiers inutiles du contexte de build : `.dockerignore` dans `services/auth-service`, `services/product-service`, `services/order-service` et `frontend`.
- Attendre MongoDB : `healthcheck` sur chaque `mongo-*` ; `auth-service`, `product-service` et `order-service` ont `depends_on` avec `condition: service_healthy` (dev et prod).
- Prod : `restart: unless-stopped` et limites `deploy.resources.limits` dans `docker-compose.prod.yml`.
- Logs : sortie standard, lecture avec `docker compose logs`.
