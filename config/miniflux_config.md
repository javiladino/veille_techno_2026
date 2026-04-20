# Guide de déploiement MiniFlux — Veille EPSI 2026

> **Rôle** : Agrégateur RSS open source, auto-hébergé, API-first  
> **Version cible** : MiniFlux 2.x (latest)  
> **Déploiement** : Docker Compose

---

## Pourquoi MiniFlux ?

MiniFlux est un lecteur RSS minimaliste et open source qui se distingue par :

- **API REST complète** : intégration native avec Make.com sans plugins tiers
- **Self-hosted** : données 100% sous contrôle, conformité RGPD garantie
- **Légèreté** : < 50 Mo RAM, idéal sur un petit VPS
- **Webhooks natifs** : envoi d'événements HTTP en temps réel sur nouveaux articles
- **Base PostgreSQL** : robuste, sauvegardable, requêtes avancées possibles

---

## Option A — Déploiement local (développement)

### Prérequis
- Docker Desktop installé
- Git installé
- Port 8080 disponible

### Installation

```bash
# 1. Cloner / créer le répertoire de configuration
mkdir miniflux-veille && cd miniflux-veille

# 2. Créer le fichier docker-compose.yml (voir contenu ci-dessous)

# 3. Lancer les conteneurs
docker compose up -d

# 4. Vérifier que les conteneurs tournent
docker compose ps

# 5. Consulter les logs
docker compose logs -f miniflux
```

### Fichier `docker-compose.yml`

```yaml
version: '3.8'

services:
  miniflux:
    image: miniflux/miniflux:latest
    container_name: miniflux_veille
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgres://miniflux:VeilleEPSI2026@db/miniflux?sslmode=disable
      RUN_MIGRATIONS: "1"
      CREATE_ADMIN: "1"
      ADMIN_USERNAME: veille_admin
      ADMIN_PASSWORD: "VeilleEPSI2026!"
      # Optionnel : activer les webhooks sortants
      POLLING_FREQUENCY: "60"
      POLLING_PARSING_ERROR_LIMIT: "3"
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "/usr/bin/miniflux", "-healthcheck", "auto"]

  db:
    image: postgres:15-alpine
    container_name: miniflux_db
    restart: unless-stopped
    environment:
      POSTGRES_USER: miniflux
      POSTGRES_PASSWORD: VeilleEPSI2026
      POSTGRES_DB: miniflux
    volumes:
      - miniflux_db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "miniflux"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  miniflux_db_data:
    driver: local
```

---

## Option B — Déploiement VPS (production)

### Prérequis VPS recommandés
- OVHcloud Starter (2 vCPU, 2 Go RAM, 20 Go SSD) ~3.5€/mois
- Scaleway DEV1-S (2 vCPU, 2 Go RAM, 20 Go SSD) ~3.99€/mois
- Ubuntu 22.04 LTS
- Docker + Docker Compose installés

### Configuration avec HTTPS (Nginx + Let's Encrypt)

```nginx
# /etc/nginx/sites-available/miniflux
server {
    listen 80;
    server_name veille.epsi.votre-domaine.fr;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name veille.epsi.votre-domaine.fr;

    ssl_certificate /etc/letsencrypt/live/veille.epsi.votre-domaine.fr/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/veille.epsi.votre-domaine.fr/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# Obtenir le certificat SSL
sudo certbot --nginx -d veille.epsi.votre-domaine.fr
```

---

## Configuration des flux RSS

### Accès à l'interface

```
URL : http://localhost:8080 (local) ou https://veille.epsi.domaine.fr (prod)
Login : veille_admin
Mot de passe : VeilleEPSI2026!
```

### Flux RSS recommandés par thématique

#### FinOps

| Source | URL du flux |
|---|---|
| FinOps Foundation | `https://www.finops.org/feed/` |
| The New Stack - Cloud | `https://thenewstack.io/feed/` |
| InfoQ Cloud | `https://feed.infoq.com/cloud` |
| Le Cloud - Le Monde Info | `https://www.lemondeinformatique.fr/flux-rss/thematique/cloud-computing/rss.xml` |
| Silicon.fr Cloud | `https://www.silicon.fr/category/cloud/feed` |

#### Green IT

| Source | URL du flux |
|---|---|
| GreenIT.fr | `https://www.greenit.fr/feed/` |
| ADEME Numérique | `https://www.ademe.fr/rss.xml` |
| WWF France | `https://www.wwf.fr/feed` |
| IEA Energy | `https://www.iea.org/rss/news.xml` |
| The Shift Project | `https://theshiftproject.org/feed/` |

#### Souveraineté des données

| Source | URL du flux |
|---|---|
| CNIL | `https://www.cnil.fr/fr/rss.xml` |
| NextINpact | `https://www.nextinpact.com/rss/articles.xml` |
| Le Monde Informatique | `https://www.lemondeinformatique.fr/flux-rss/rss.xml` |
| Silicon.fr | `https://www.silicon.fr/feed` |
| Numerama | `https://www.numerama.com/feed/` |

#### Sources académiques & institutionnelles

| Source | URL du flux |
|---|---|
| INRIA Actualités | `https://www.inria.fr/fr/rss.xml` |
| European Commission Digital | `https://digital-strategy.ec.europa.eu/en/rss.xml` |
| ENISA News | `https://www.enisa.europa.eu/rss-feeds/rss.xml` |

### Import en masse via OPML

```xml
<!-- miniflux_feeds_v2.opml — à importer dans MiniFlux -->
<?xml version="1.0" encoding="UTF-8"?>
<opml version="2.0">
  <head>
    <title>Veille EPSI 2026 — Flux RSS</title>
    <dateCreated>2026-04-18</dateCreated>
  </head>
  <body>
    <outline text="FinOps" title="FinOps">
      <outline type="rss" text="FinOps Foundation" title="FinOps Foundation"
               xmlUrl="https://www.finops.org/feed/" htmlUrl="https://www.finops.org"/>
      <outline type="rss" text="The New Stack" title="The New Stack"
               xmlUrl="https://thenewstack.io/feed/" htmlUrl="https://thenewstack.io"/>
    </outline>
    <outline text="Green IT" title="Green IT">
      <outline type="rss" text="GreenIT.fr" title="GreenIT.fr"
               xmlUrl="https://www.greenit.fr/feed/" htmlUrl="https://www.greenit.fr"/>
      <outline type="rss" text="ADEME" title="ADEME"
               xmlUrl="https://www.ademe.fr/rss.xml" htmlUrl="https://www.ademe.fr"/>
    </outline>
    <outline text="Souveraineté" title="Souveraineté des données">
      <outline type="rss" text="CNIL" title="CNIL"
               xmlUrl="https://www.cnil.fr/fr/rss.xml" htmlUrl="https://www.cnil.fr"/>
      <outline type="rss" text="NextINpact" title="NextINpact"
               xmlUrl="https://www.nextinpact.com/rss/articles.xml" htmlUrl="https://www.nextinpact.com"/>
    </outline>
  </body>
</opml>
```

---

## Configuration de l'API MiniFlux

### Obtenir la clé API

```bash
# Via l'interface web
→ Réglages → API Keys → Créer une nouvelle clé API
→ Nom : "make_com_integration"
→ Copier la clé générée

# Via CLI (si accès Docker)
docker exec miniflux_veille /usr/bin/miniflux -create-admin
```

### Tester l'API

```bash
# Variables d'environnement
export MINIFLUX_URL="http://localhost:8080"
export MINIFLUX_API_KEY="votre-clé-api-ici"

# Lister les articles non lus
curl -s -H "X-Auth-Token: $MINIFLUX_API_KEY" \
  "$MINIFLUX_URL/v1/entries?status=unread&limit=10" | python3 -m json.tool

# Lister les flux configurés
curl -s -H "X-Auth-Token: $MINIFLUX_API_KEY" \
  "$MINIFLUX_URL/v1/feeds" | python3 -m json.tool

# Marquer un article comme lu
curl -s -X PUT -H "X-Auth-Token: $MINIFLUX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"entry_ids": [123], "status": "read"}' \
  "$MINIFLUX_URL/v1/entries"
```

### Endpoints utiles pour Make.com

| Endpoint | Méthode | Usage |
|---|---|---|
| `/v1/entries?status=unread&limit=50` | GET | Récupérer nouveaux articles |
| `/v1/entries?status=unread&starred=true` | GET | Articles favoris |
| `/v1/feeds` | GET | Liste de tous les flux |
| `/v1/entries/{id}` | PUT | Marquer comme lu |
| `/v1/entries?category_id=X` | GET | Filtrer par catégorie |

---

## Configuration des webhooks MiniFlux

MiniFlux supporte les webhooks natifs pour envoyer des événements HTTP à Make.com en temps réel (alternative au polling).

### Activer dans docker-compose.yml

```yaml
environment:
  # Ajouter ces variables
  WATCHDOG_WORKER_POOL_SIZE: "1"
```

### Configuration webhook dans l'interface

```
Réglages → Intégrations → Webhook
URL : https://hook.eu2.make.com/[votre-id-webhook]
Événements : new_entries (nouveaux articles)
Secret : [votre-secret-make-com]
```

---

## Sauvegarde et maintenance

### Sauvegarde de la base de données

```bash
# Sauvegarde manuelle
docker exec miniflux_db pg_dump -U miniflux miniflux > backup_$(date +%Y%m%d).sql

# Sauvegarde automatique (cron hebdomadaire)
0 3 * * 0 docker exec miniflux_db pg_dump -U miniflux miniflux > /backups/miniflux_$(date +\%Y\%m\%d).sql
```

### Mise à jour MiniFlux

```bash
# Mettre à jour vers la dernière version
docker compose pull miniflux
docker compose up -d miniflux
docker compose logs -f miniflux
```

### Monitoring basique

```bash
# Vérifier l'état des conteneurs
docker compose ps

# Utilisation des ressources
docker stats miniflux_veille miniflux_db

# Espace disque base de données
docker exec miniflux_db psql -U miniflux -c "SELECT pg_size_pretty(pg_database_size('miniflux'));"
```

---

*Configuration MiniFlux — Projet veille technologique EPSI 2025-2026*
