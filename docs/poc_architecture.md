# Architecture & Pipeline de données — POC v2

> **Projet** : Système de Veille Technologique EPSI 2026
> **Stack** : Miniflux · n8n · Google Drive · Slack · NotebookLM
> **Infrastructure** : VM Proxmox (Debian 12) — Self-hosted natif, sans Docker

---

## 1. Architecture globale

```mermaid
flowchart TD
    subgraph SOURCES["📡 SOURCES RSS"]
        S1["💰 FinOps\nfinops.org · lemondeinformatique.fr\nsilicon.fr · thenewstack.io"]
        S2["🌿 Green IT\ngreenit.fr · theshiftproject.org\nademe.fr · next.ink"]
        S3["🔐 Souveraineté\ncnil.fr · next.ink\nsilicon.fr · numerama.com"]
    end

    subgraph VM["🖥️ VM Proxmox — Debian 12 (i2-infra-veille-g1)"]
        subgraph MINIFLUX["Miniflux — Port 8080"]
            MF["Moteur RSS\nService systemd"]
            DB["PostgreSQL 15\nPort 5432"]
            API["API REST\n/v1/entries"]
            MF --> DB
            MF --> API
        end
        subgraph N8N["n8n — Port 5678"]
            WF1["Workflow 1\nCollecte (2h)"]
            WF2["Workflow 2\nDigest (lundi 8h)"]
            WF3["Workflow 3\nPodcast (trigger)"]
        end
        NGINX["Nginx — Port 80\nReverse Proxy"]
        NGINX --> MF
    end

    subgraph GDRIVE["💾 Google Drive"]
        RAW["📁 /01_Raw/\narticle_AAAA-MM-JJ_ID.md"]
        DIGEST["📁 /02_Digest/\ndigest_semaine_XX.md"]
        PODCAST["📁 /03_Podcast/\npodcast_semaine_XX.m4a"]
        REPORTS["📁 /04_Reports/\nrapport_mensuel.md"]
    end

    subgraph SLACK["📢 Slack"]
        SL1["#veille-critique\n🚨 Temps réel"]
        SL2["#veille-hebdo\n📊 Lundi 08h"]
        SL3["#veille-podcast\n🎙️ À la détection"]
    end

    subgraph NLM["🎙️ NotebookLM"]
        NB["Notebook\nVeille EPSI 2026"]
        AUDIO["Audio Overview\n~10 min génération"]
        M4A["Fichier .m4a\n15-20 min"]
        NB --> AUDIO --> M4A
    end

    S1 & S2 & S3 -->|"RSS/Atom"| MINIFLUX
    API -->|"JSON"| WF1
    API -->|"JSON"| WF2
    WF1 -->|"Upload .md"| RAW
    WF1 -->|"Webhook"| SL1
    WF2 -->|"Upload .md"| DIGEST
    WF2 -->|"Webhook"| SL2
    DIGEST -->|"Source manuelle"| NB
    M4A -->|"Dépôt manuel"| PODCAST
    WF3 -->|"Webhook"| SL3
    PODCAST -->|"Trigger"| WF3

    classDef vm fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    classDef mf fill:#e8f8e8,stroke:#27ae60,stroke-width:2px
    classDef n8n fill:#fff3e0,stroke:#f39c12,stroke-width:2px
    classDef storage fill:#f3e5f5,stroke:#8e44ad,stroke-width:2px
    classDef comms fill:#e0f2f1,stroke:#16a085,stroke-width:2px
    classDef podcast fill:#fce4ec,stroke:#e74c3c,stroke-width:2px
    classDef sources fill:#e8f4f8,stroke:#2980b9,stroke-width:2px

    class VM,NGINX vm
    class MF,DB,API mf
    class WF1,WF2,WF3 n8n
    class RAW,DIGEST,PODCAST,REPORTS storage
    class SL1,SL2,SL3 comms
    class NB,AUDIO,M4A podcast
    class S1,S2,S3 sources
```

---

## 2. Workflow 1 — Collecte des articles

**Déclenchement** : toutes les 2 heures, 24h/24

```mermaid
flowchart LR
    T1["⏱️ Schedule\nTrigger\n2h"] --> H1
    H1["🌐 HTTP Request\nGET /v1/entries\n?status=unread"] --> SO
    SO["✂️ Split Out\nSéparer les\nentrées"] --> CF
    CF{"🔎 Code\nFiltrage\nkeywords"} -->|"Match"| CB
    CF -->|"No match"| END1["🚫 Stop"]
    CB["📝 Code\nConstruction\nmarkdown .md"] --> GD
    GD["☁️ Google Drive\nUpload\n/01_Raw/"] --> MR
    MR["✅ HTTP Request\nPUT /v1/entries\nMarquer lu"] --> SL
    SL["💬 Slack\n#veille-critique\nWebhook"]

    style T1 fill:#e3f2fd
    style CF fill:#fff3e0
    style GD fill:#f3e5f5
    style SL fill:#e0f2f1
```

**Mots-clés de filtrage :**

```javascript
// FinOps
const finopsKw = ['finops', 'cloud cost', 'tco', 'cost optimization',
  'capex', 'opex', 'cloud billing', 'coût du cloud', 'budget cloud'];

// Green IT
const greenKw = ['green it', 'énergie', 'datacenter', 'empreinte carbone',
  'sustainability', 'carbon', 'renewable energy', 'sobriété numérique',
  'consommation énergétique', 'transition écologique'];

// Souveraineté
const sovKw = ['sovereignty', 'souveraineté', 'gdpr', 'rgpd',
  'cloud repatriation', 'ai act', 'données personnelles',
  'conformité', 'compliance', 'vie privée', 'réglementation'];
```

---

## 3. Workflow 2 — Digest hebdomadaire

**Déclenchement** : chaque lundi à 08h00

```mermaid
flowchart LR
    T2["⏱️ Schedule\nLundi 08h\nCron"] --> CA
    CA["🔢 Code\nTimestamp -7j\nN° semaine"] --> H2
    H2["🌐 HTTP Request\nGET /v1/entries\n?after=TIMESTAMP"] --> CD
    CD["📊 Code\nAgrégation\npar thématique"] --> GD2
    GD2["☁️ Google Drive\nUpload\n/02_Digest/"] --> SL2
    SL2["💬 Slack\n#veille-hebdo\n+ action NLM"]

    style T2 fill:#e3f2fd
    style CD fill:#fff3e0
    style GD2 fill:#f3e5f5
    style SL2 fill:#e0f2f1
```

**Structure du digest généré :**

```text
# Digest Veille Technologique — Semaine XX
**Total articles** : N

## 💰 FinOps (N articles)
### Titre article
**Source** : X | **Date** : AAAA-MM-JJ
**URL** : https://...
Extrait...

## 🌿 Green IT (N articles)
## 🔐 Souveraineté des données (N articles)
## 📌 Autres (N articles)
```

---

## 4. Workflow 3 — Notification podcast

**Déclenchement** : création d'un fichier dans `/03_Podcast/`

```mermaid
flowchart LR
    GT["📁 Google Drive\nTrigger\n/03_Podcast/"] --> SL3
    SL3["💬 Slack\n#veille-podcast\nLien + nom fichier"]

    style GT fill:#f3e5f5
    style SL3 fill:#e0f2f1
```

---

## 5. Séquence temporelle hebdomadaire

```mermaid
sequenceDiagram
    participant RSS as Sources RSS
    participant MF as Miniflux (VM)
    participant N8N as n8n (VM)
    participant GD as Google Drive
    participant NLM as NotebookLM
    participant SL as Slack

    Note over RSS,SL: 🔄 Cycle continu — toutes les 2 heures

    RSS->>MF: Nouveaux articles (RSS/Atom)
    MF->>MF: Indexation PostgreSQL
    N8N->>MF: GET /v1/entries?status=unread
    MF-->>N8N: JSON articles
    N8N->>N8N: Filtrage mots-clés
    N8N->>GD: Upload article_AAAA-MM-JJ_ID.md → /01_Raw/
    N8N->>MF: PUT /v1/entries (marquer lu)
    N8N->>SL: POST #veille-critique

    Note over N8N,SL: 📅 Lundi 08h00 — Digest hebdomadaire

    N8N->>MF: GET /v1/entries?after=7jours
    MF-->>N8N: JSON articles semaine
    N8N->>N8N: Agrégation par thématique
    N8N->>GD: Upload digest_semaine_XX.md → /02_Digest/
    N8N->>SL: POST #veille-hebdo (+ instruction NotebookLM)

    Note over GD,NLM: 📅 Vendredi — Production podcast (manuel ~4 min)

    GD-->>NLM: Source : digest_semaine_XX.md (ajout manuel)
    NLM->>NLM: Génération Audio Overview (~10 min)
    NLM-->>GD: Dépôt manuel .m4a → /03_Podcast/
    GD->>N8N: Trigger (nouveau fichier détecté)
    N8N->>SL: POST #veille-podcast (lien écoute)
```

---

## 6. Infrastructure physique

```text
┌─────────────────────────────────────────────────────────────────────┐
│                    PROXMOX — Hyperviseur on-premise                  │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  VM : i2-infra-veille-g1                                      │   │
│  │  OS : Debian 12.4 (Bookworm)                                  │   │
│  │  IP : 172.16.89.44/16  |  Gateway : 172.16.255.254            │   │
│  │  Ressources : 2 vCPU · 2 Go RAM · 20 Go disque               │   │
│  │                                                               │   │
│  │  Services actifs :                                            │   │
│  │  ┌───────────────────┐  ┌────────────────────────────────┐   │   │
│  │  │  PostgreSQL 15    │  │  Miniflux (binaire natif)      │   │   │
│  │  │  Port 5432        │◄─│  Service : miniflux.service    │   │   │
│  │  │  DB : miniflux    │  │  Config : /etc/miniflux.conf   │   │   │
│  │  │  User : miniflux  │  │  Port : 127.0.0.1:8080         │   │   │
│  │  └───────────────────┘  └────────────────────────────────┘   │   │
│  │                                        ▲                      │   │
│  │  ┌─────────────────────────────────────┘                     │   │
│  │  │  Nginx (reverse proxy)                                     │   │
│  │  │  Port 80 → 127.0.0.1:8080                                 │   │
│  │  │  Config : /etc/nginx/sites-available/miniflux              │   │
│  │  └───────────────────────────────────────────────────────    │   │
│  │                                                               │   │
│  │  ┌──────────────────────────────────────────────────────┐   │   │
│  │  │  n8n (npm global)                                     │   │   │
│  │  │  Service : n8n.service                                │   │   │
│  │  │  Port : 127.0.0.1:5678                                │   │   │
│  │  │  WEBHOOK_URL : http://localhost:5678/                 │   │   │
│  │  └──────────────────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                       │
│  Accès poste client :                                                │
│  ssh -L 8080:127.0.0.1:8080 jladino@172.16.89.44  → Miniflux        │
│  ssh -L 5678:127.0.0.1:5678 jladino@172.16.89.44  → n8n             │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS sortant (port 443)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      SERVICES CLOUD (SaaS)                           │
│                                                                       │
│  Google Drive               Slack                NotebookLM          │
│  /01_Raw/                   #veille-critique     Notebook            │
│  /02_Digest/                #veille-hebdo        Veille EPSI 2026    │
│  /03_Podcast/               #veille-podcast      Audio Overview      │
│  /04_Reports/                                                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. Modèle de données

```text
Article (01_Raw)
├── date            : Date (AAAA-MM-JJ)
├── source          : String (nom du flux RSS)
├── titre           : String
├── url             : String (URL canonique)
├── tags            : Array[String]
└── résumé          : Text (extrait RSS, HTML strippé)

Digest (02_Digest)
├── semaine         : Integer (numéro ISO)
├── date            : Date
├── total_articles  : Integer
├── finops          : Array[Article] (max 5)
├── greenit         : Array[Article] (max 5)
├── sovereignty     : Array[Article] (max 5)
└── autres          : Array[Article] (max 5)

Podcast (03_Podcast)
├── semaine         : Integer
├── format          : String (.m4a)
├── durée           : Float (~15-20 min)
└── source_digest   : String (référence digest)
```

---

## 8. Matrice de décision infrastructurelle

| Dimension | VM Proxmox (v2) | VPS Cloud | Docker local |
|---|---|---|---|
| **Souveraineté** | ✅ Totale | ⚠️ Cloud tiers | ✅ Locale |
| **RGPD** | ✅ Conforme | ⚠️ CGU hébergeur | ✅ Conforme |
| **Coût** | ✅ 0€ | ~5€/mois | ✅ 0€ |
| **Disponibilité** | Proxmox uptime | 99.9% SLA | PC allumé |
| **Accès externe** | SSH tunnel | Direct HTTPS | Tunnel requis |
| **Maintenance** | APT updates | APT/Docker | Docker updates |
| **Scalabilité** | Selon Proxmox | Illimitée | Limité |

---

*Architecture rédigée pour le projet de veille technologique — Master EPSI 2025-2026*
*Infrastructure : VM Proxmox · Miniflux natif · n8n · Google Drive · Slack · NotebookLM*
