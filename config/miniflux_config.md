# Guide de déploiement Miniflux — Debian 12 (Proxmox)

> **Rôle** : Agrégateur RSS open source, auto-hébergé, API-first
> **Version** : Miniflux 2.x (dépôt officiel APT)
> **Déploiement** : Binaire natif — service systemd, sans Docker
> **Environnement** : VM Proxmox · Debian 12.4 Bookworm

---

## Prérequis

- VM Debian 12 provisionnée dans Proxmox
- Accès SSH ou console Proxmox
- Connexion internet depuis la VM (DNS public configuré)
- Utilisateur avec droits `sudo`

---

## 1. Préparation du système

```bash
# Mise à jour du système
sudo apt update && sudo apt upgrade -y

# Dépendances de base
sudo apt install -y curl gnupg ca-certificates nginx

# Vérifier la connectivité DNS (requis pour les dépôts)
ping -c 2 deb.debian.org
```

> **Note DNS** : Si la résolution DNS échoue, ajouter un serveur public dans `/etc/resolv.conf` :
>
> ```bash
> sudo nano /etc/resolv.conf
> # Ajouter en première ligne :
> nameserver 8.8.8.8
> ```
>
> Pour rendre la configuration persistante via DHCP :
>
> ```bash
> echo "prepend domain-name-servers 8.8.8.8;" | sudo tee -a /etc/dhcp/dhclient.conf
> ```

---

## 2. Installation de PostgreSQL

```bash
sudo apt install -y postgresql postgresql-contrib
sudo systemctl enable postgresql
sudo systemctl start postgresql

# Vérification
sudo systemctl status postgresql
```

---

## 3. Création de la base de données

```bash
# Créer l'utilisateur Miniflux
sudo -u postgres createuser -P miniflux
# → Saisir un mot de passe fort (sans caractères spéciaux shell)

# Créer la base de données
sudo -u postgres createdb -O miniflux miniflux

# Activer l'extension hstore
sudo -u postgres psql miniflux -c 'create extension hstore'
```

> **Résolution d'erreur** : Si `must be owner of extension hstore` lors de la migration :
>
> ```bash
> sudo -u postgres psql -c "ALTER USER miniflux WITH SUPERUSER;"
> sudo miniflux -config-file /etc/miniflux.conf -migrate
> sudo -u postgres psql -c "ALTER USER miniflux WITH NOSUPERUSER;"
> ```

---

## 4. Installation de Miniflux

```bash
# Télécharger le paquet .deb depuis GitHub Releases
DEB_URL=$(curl -s https://api.github.com/repos/miniflux/v2/releases/latest \
  | grep "browser_download_url.*amd64.deb" \
  | cut -d '"' -f 4)

curl -L -o miniflux.deb "$DEB_URL"
sudo dpkg -i miniflux.deb
sudo apt install -f  # Résoudre les dépendances si nécessaire
```

---

## 5. Configuration

```bash
sudo nano /etc/miniflux.conf
```

Contenu du fichier `/etc/miniflux.conf` :

```ini
# Connexion PostgreSQL
DATABASE_URL=postgres://miniflux:MOT_DE_PASSE@localhost/miniflux?sslmode=disable

# Adresse d'écoute interne (Nginx forward vers ce port)
LISTEN_ADDR=127.0.0.1:8080

# URL publique de l'instance (utilisée pour la validation CSRF)
BASE_URL=http://localhost/

# Clé secrète pour les sessions (générer avec : openssl rand -hex 32)
SECRET_KEY=VALEUR_GENEREE

# Intervalle de polling des flux RSS (en minutes)
POLLING_FREQUENCY=30

# Niveau de journalisation
LOG_LEVEL=info
```

Générer la `SECRET_KEY` :

```bash
openssl rand -hex 32
```

---

## 6. Initialisation et création de l'administrateur

```bash
# Exécuter les migrations de base de données
sudo miniflux -config-file /etc/miniflux.conf -migrate

# Créer le compte administrateur
sudo miniflux -config-file /etc/miniflux.conf -create-admin
```

---

## 7. Activation du service systemd

```bash
sudo systemctl enable miniflux
sudo systemctl start miniflux

# Vérification
sudo systemctl status miniflux
sudo journalctl -u miniflux -n 20 --no-pager
```

---

## 8. Configuration Nginx (reverse proxy)

```bash
sudo nano /etc/nginx/sites-available/miniflux
```

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass         http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header   Host              $host;
        proxy_set_header   X-Forwarded-Host  $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/miniflux /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl enable nginx
```

---

## 9. Accès depuis le poste client

Miniflux écoute sur `127.0.0.1` uniquement. L'accès se fait via tunnel SSH :

```bash
# Depuis le poste client (Mac/Linux)
ssh -L 8080:127.0.0.1:8080 jladino@172.16.89.44

# Accès dans le navigateur
http://localhost:8080/
```

---

## 10. Configuration de l'API pour n8n

Dans l'interface Miniflux :

1. **Settings → API Keys → Create a new API key**
2. Nom : `n8n_veille`
3. Copier la clé générée

### Endpoints API utilisés par n8n

| Endpoint | Méthode | Usage |
|---|---|---|
| `/v1/entries?status=unread&limit=50` | GET | Collecte articles non lus |
| `/v1/entries?after=TIMESTAMP&limit=100` | GET | Articles des 7 derniers jours |
| `/v1/entries` | PUT | Marquer articles comme lus |
| `/v1/feeds` | GET | Lister les flux configurés |

### Test de l'API

```bash
# Depuis la VM (connexion locale directe)
curl -s http://127.0.0.1:8080/v1/me \
  -H "X-Auth-Token: VOTRE_CLE_API" | python3 -m json.tool
```

---

## 11. Flux RSS configurés

### FinOps

| Source | URL |
|---|---|
| FinOps Foundation | `https://www.finops.org/feed/` |
| Le Monde Informatique — Infrastructure | `https://www.lemondeinformatique.fr/flux-rss/thematique/infrastructure/rss.xml` |
| Silicon.fr | `https://www.silicon.fr/feed` |
| The New Stack | `https://thenewstack.io/feed/` |

### Green IT

| Source | URL |
|---|---|
| GreenIT.fr | `https://www.greenit.fr/feed/` |
| The Shift Project | `https://theshiftproject.org/feed/` |
| ADEME | `https://www.ademe.fr/rss.xml` |
| Next.ink | `https://next.ink/feed/` |

### Souveraineté des données

| Source | URL |
|---|---|
| CNIL | `https://www.cnil.fr/fr/rss.xml` |
| Next.ink | `https://next.ink/feed/` |
| Silicon.fr | `https://www.silicon.fr/feed` |
| Numerama | `https://www.numerama.com/feed/` |

---

## 12. Maintenance

### Mise à jour de Miniflux

```bash
# Télécharger et installer la nouvelle version
DEB_URL=$(curl -s https://api.github.com/repos/miniflux/v2/releases/latest \
  | grep "browser_download_url.*amd64.deb" \
  | cut -d '"' -f 4)

curl -L -o miniflux_new.deb "$DEB_URL"
sudo systemctl stop miniflux
sudo dpkg -i miniflux_new.deb
sudo miniflux -config-file /etc/miniflux.conf -migrate
sudo systemctl start miniflux
```

### Sauvegarde PostgreSQL

```bash
# Sauvegarde manuelle
sudo -u postgres pg_dump miniflux > backup_miniflux_$(date +%Y%m%d).sql

# Sauvegarde automatique hebdomadaire (cron)
# sudo crontab -e
0 3 * * 0 sudo -u postgres pg_dump miniflux > /var/backups/miniflux_$(date +\%Y\%m\%d).sql
```

### Vérification de l'état des services

```bash
sudo systemctl status miniflux postgresql nginx
sudo journalctl -u miniflux -n 50 --no-pager
```

### Réinitialisation du mot de passe admin

```bash
sudo miniflux -config-file /etc/miniflux.conf -reset-password
```

---

*Guide de déploiement Miniflux — Projet veille technologique EPSI 2025-2026*
*Infrastructure : VM Proxmox · Debian 12 Bookworm · Binaire natif*
