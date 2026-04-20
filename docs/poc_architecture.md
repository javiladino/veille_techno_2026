# Architecture & Pipeline de données — Version POC v2

> **Projet** : Système de Veille Technologique EPSI 2026  
> **Stack** : MiniFlux · Make.com · Google Drive · Slack · NotebookLM  
> **Document** : Schéma d'infrastructure et flux de données

---

## 1. Schéma d'infrastructure global (Mermaid)

```mermaid
flowchart TD
    subgraph SOURCES["📡 SOURCES D'INFORMATION"]
        direction TB
        S1["🔵 FinOps\nFinOps Foundation, Cloud Cost News\nGartner, Forrester"]
        S2["🟢 Green IT\ngreenit.fr, IEA, WWF Digital\nINRIA, Ademe"]
        S3["🔴 Souveraineté\nCNIL, Silicon.fr\nLe Monde Informatique\nNextINpact"]
    end

    subgraph COLLECTE["🔍 COLLECTE — MiniFlux (Self-hosted)"]
        direction TB
        MF_ENGINE["MiniFlux Engine\nDocker · Port 8080"]
        MF_DB["PostgreSQL\nBase de données locale"]
        MF_API["API REST\n/v1/entries\n/v1/feeds"]
        MF_ENGINE --> MF_DB
        MF_ENGINE --> MF_API
    end

    subgraph AUTOMATION["⚙️ AUTOMATISATION — Make.com"]
        direction TB
        TRIGGER["🔄 Trigger\nHTTP Polling MiniFlux\nToutes les 2 heures"]
        FILTER["🔎 Filtre\nMots-clés thématiques\nFinOps · GreenIT · Souveraineté"]
        ROUTER{"🔀 Routeur\nScore de pertinence"}
        BRANCH_CRIT["⚡ Branche Critique\nScore ≥ 8/10"]
        BRANCH_STD["📄 Branche Standard\nTous articles"]
        BRANCH_HEBDO["📊 Branche Hebdo\nLundi 08h00"]
        BRANCH_PODCAST["🎙️ Branche Podcast\nVendredi 16h00"]

        TRIGGER --> FILTER --> ROUTER
        ROUTER --> BRANCH_CRIT
        ROUTER --> BRANCH_STD
        ROUTER --> BRANCH_HEBDO
        ROUTER --> BRANCH_PODCAST
    end

    subgraph STORAGE["💾 STOCKAGE — Google Drive"]
        direction TB
        GD_RAW["📁 /Raw/\nArticles bruts\nformat Markdown"]
        GD_DIGEST["📁 /Digest/\nRésumés hebdomadaires\nTop 10 articles"]
        GD_PODCAST["📁 /Podcast/\nFichiers MP3\npodcast_semaine_XX"]
        GD_REPORTS["📁 /Reports/\nRapports mensuels\nTableau de bord"]
        GD_SHEETS["📊 suivi_articles.gsheet\nKPIs & métriques"]
    end

    subgraph PODCAST_ENGINE["🎙️ PRODUCTION PODCAST — NotebookLM"]
        direction TB
        NLM_NB["Notebook\nVeille EPSI 2026"]
        NLM_SRC["Sources permanentes\nGlossaires · Contexte"]
        NLM_WEEKLY["Source hebdo\nDigest semaine"]
        NLM_AUDIO["Audio Overview\nGénération automatique"]
        NLM_MP3["Podcast MP3\n15-20 minutes"]

        NLM_NB --> NLM_SRC
        NLM_NB --> NLM_WEEKLY
        NLM_NB --> NLM_AUDIO --> NLM_MP3
    end

    subgraph COMMS["📢 COMMUNICATION — Slack"]
        direction TB
        SLACK1["#veille-critique\n🚨 Alertes temps réel"]
        SLACK2["#veille-hebdo\n📊 Digest du lundi"]
        SLACK3["#veille-podcast\n🎙️ Podcast du vendredi"]
    end

    %% Flux de données
    S1 --> |"RSS/Atom feeds"| COLLECTE
    S2 --> |"RSS/Atom feeds"| COLLECTE
    S3 --> |"RSS/Atom feeds"| COLLECTE

    MF_API --> |"JSON entries"| TRIGGER

    BRANCH_CRIT --> SLACK1
    BRANCH_STD --> GD_RAW
    BRANCH_HEBDO --> GD_DIGEST
    BRANCH_HEBDO --> SLACK2
    BRANCH_PODCAST --> NLM_WEEKLY

    GD_DIGEST --> |"Upload source"| NLM_WEEKLY
    NLM_MP3 --> |"Sauvegarde"| GD_PODCAST
    GD_PODCAST --> |"Lien partage"| SLACK3

    GD_RAW --> GD_SHEETS
    GD_DIGEST --> GD_REPORTS

    %% Styles
    classDef sources fill:#e8f4f8,stroke:#2980b9,stroke-width:2px
    classDef collecte fill:#e8f8e8,stroke:#27ae60,stroke-width:2px
    classDef auto fill:#fff3e0,stroke:#f39c12,stroke-width:2px
    classDef storage fill:#f3e5f5,stroke:#8e44ad,stroke-width:2px
    classDef podcast fill:#fce4ec,stroke:#e74c3c,stroke-width:2px
    classDef comms fill:#e0f2f1,stroke:#16a085,stroke-width:2px

    class S1,S2,S3 sources
    class MF_ENGINE,MF_DB,MF_API collecte
    class TRIGGER,FILTER,ROUTER,BRANCH_CRIT,BRANCH_STD,BRANCH_HEBDO,BRANCH_PODCAST auto
    class GD_RAW,GD_DIGEST,GD_PODCAST,GD_REPORTS,GD_SHEETS storage
    class NLM_NB,NLM_SRC,NLM_WEEKLY,NLM_AUDIO,NLM_MP3 podcast
    class SLACK1,SLACK2,SLACK3 comms
```

---

## 2. Diagramme de séquence temporelle

```mermaid
sequenceDiagram
    participant RSS as Sources RSS
    participant MF as MiniFlux
    participant MC as Make.com
    participant GD as Google Drive
    participant NLM as NotebookLM
    participant SL as Slack

    Note over RSS,SL: 🔄 Cycle continu — toutes les 2 heures

    RSS->>MF: Nouveaux articles (RSS/Atom)
    MF->>MF: Indexation + stockage PostgreSQL
    MC->>MF: Polling API /v1/entries?status=unread
    MF-->>MC: JSON liste d'articles

    MC->>MC: Filtrage mots-clés thématiques
    MC->>MC: Calcul score de pertinence

    alt Article critique (score ≥ 8)
        MC->>SL: Post #veille-critique (temps réel)
        MC->>GD: Sauvegarde /Raw/ + marquage critique
    else Article standard
        MC->>GD: Sauvegarde /Raw/
        MC->>GD: Mise à jour suivi_articles.gsheet
    end

    Note over MC,SL: 📅 Lundi 08h00 — Digest hebdomadaire

    MC->>GD: Lire articles semaine depuis /Raw/
    MC->>GD: Générer digest_semaine_XX.md → /Digest/
    MC->>SL: Post #veille-hebdo avec top 5 articles

    Note over MC,NLM: 📅 Vendredi 16h00 — Production podcast

    MC->>GD: Lire digest_semaine_XX.md
    MC->>NLM: Upload digest comme nouvelle source
    NLM->>NLM: Traitement Audio Overview (15 min)
    NLM-->>MC: Podcast MP3 disponible
    MC->>GD: Sauvegarde /Podcast/podcast_semaine_XX.mp3
    MC->>SL: Post #veille-podcast avec lien Drive

    Note over MC,SL: 📅 1er du mois — Rapport mensuel

    MC->>GD: Agréger données du mois
    MC->>GD: Créer rapport_mensuel_MOIS_2026.md
    MC->>SL: Notification équipe rapport disponible
```

---

## 3. Schéma d'infrastructure physique

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ENVIRONNEMENT DE DÉPLOIEMENT                      │
│                                                                       │
│  Option A — Local (Dev)          Option B — VPS (Production)         │
│  ┌──────────────────────┐        ┌──────────────────────────────┐   │
│  │  MacBook / PC        │        │  VPS OVHcloud / Scaleway     │   │
│  │  Docker Desktop      │        │  2 vCPU · 4 Go RAM · 40 Go   │   │
│  │  ┌────────────────┐  │        │  Ubuntu 22.04 LTS            │   │
│  │  │  miniflux:8080 │  │        │  ┌──────────────────────┐    │   │
│  │  │  postgres:5432 │  │        │  │  miniflux:8080       │    │   │
│  │  └────────────────┘  │        │  │  postgres:5432       │    │   │
│  │  Accès : localhost   │        │  │  nginx:80/443 (HTTPS)│    │   │
│  └──────────────────────┘        │  └──────────────────────┘    │   │
│                                  │  Accès : https://veille.epsi  │   │
│                                  └──────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ API REST HTTPS
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      SERVICES CLOUD (SaaS)                           │
│                                                                       │
│   Make.com              Google Drive            Slack                │
│   ┌──────────────┐      ┌──────────────┐       ┌──────────────┐    │
│   │ Scénario v2  │─────►│ /Raw/        │       │ #critique    │    │
│   │ 3 branches   │      │ /Digest/     │       │ #hebdo       │    │
│   │ Polling 2h   │      │ /Podcast/    │       │ #podcast     │    │
│   └──────────────┘      │ /Reports/    │       └──────────────┘    │
│          │              └──────────────┘                            │
│          │                     │                                     │
│          │              ┌──────▼──────┐                             │
│          └─────────────►│  NotebookLM │                             │
│                         │  Notebook   │                             │
│                         │  EPSI 2026  │                             │
│                         └─────────────┘                             │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Modèle de données — Article de veille

```
Article (entité principale)
├── id              : UUID (généré MiniFlux)
├── titre           : String
├── url             : String (URL canonique)
├── date_publication: DateTime
├── date_collecte   : DateTime (timestamp MiniFlux)
├── source_name     : String (nom du flux RSS)
├── source_url      : String (URL du flux)
├── contenu_html    : Text (article complet si disponible)
├── extrait         : Text (résumé RSS, max 500 chars)
├── tags_auto       : Array[String] (détectés par filtre Make.com)
├── thematique      : Enum[FinOps, GreenIT, Souveraineté, Hybride, Autre]
├── score_pertinence: Integer (1-10, calculé par Make.com)
├── statut          : Enum[nouveau, analysé, archivé]
└── path_drive      : String (chemin Google Drive)

Digest hebdomadaire
├── semaine         : Integer (numéro ISO semaine)
├── annee           : Integer
├── date_debut      : Date
├── date_fin        : Date
├── articles_count  : Integer
├── top_articles    : Array[Article.id] (max 10)
├── resume_finops   : Text
├── resume_greenit  : Text
├── resume_souv     : Text
├── tendance_semaine: Text
└── path_drive      : String

Podcast
├── semaine         : Integer
├── annee           : Integer
├── duree_minutes   : Float
├── digest_source   : Digest.id (référence)
├── notebooklm_nb   : String (ID notebook NotebookLM)
├── path_drive      : String (chemin MP3)
└── url_slack       : String (lien message Slack)
```

---

## 5. Matrice de décision infrastructurelle

| Dimension | MiniFlux Self-hosted | Feedly (v1) | Talkwalker (v1) |
|---|---|---|---|
| **Souveraineté** | ✅ Totale | ⚠️ Cloud US | ⚠️ Cloud US |
| **RGPD** | ✅ Conforme | ⚠️ CGU tiers | ⚠️ CGU tiers |
| **Coût** | ~5€/mois (VPS) | 7-18€/mois | 50€+/mois |
| **Maintenance** | Docker updates | Aucune | Aucune |
| **API** | ✅ REST complète | Limitée (tier) | Limitée (tier) |
| **Contrôle données** | ✅ Total | ❌ Aucun | ❌ Aucun |
| **Scalabilité** | Selon VPS | Illimitée | Illimitée |
| **Fonctionnalités** | RSS/Atom core | RSS + curation | Social + web |

---

*Architecture rédigée pour le projet de veille technologique — Master EPSI 2025-2026*
