# Système de Veille Technologique — POC v2

> **Projet** : Veille Technologique — Master EPSI 2025-2026  
> **Version** : 2.0 — Proof of Concept (déployé et opérationnel)  
> **Équipe** : Serge WEMBE II-ESSOUMBA · Cheik LAWANI · Javier LADINO  
> **Thématiques** : FinOps · Green IT · Souveraineté des données

---

## Sommaire

1. [Contexte et motivation](#1-contexte-et-motivation)
2. [Problématique centrale](#2-problématique-centrale)
3. [Stack technique v2](#3-stack-technique-v2)
4. [Architecture et infrastructure](#4-architecture-et-infrastructure)
5. [Workflows automatisés](#5-workflows-automatisés)
6. [Organisation Google Drive](#6-organisation-google-drive)
7. [Communication Slack](#7-communication-slack)
8. [Production du podcast — NotebookLM](#8-production-du-podcast--notebooklm)
9. [Comparatif v1 vs v2](#9-comparatif-v1-vs-v2)
10. [KPIs et métriques](#10-kpis-et-métriques)
11. [Structure du dépôt](#11-structure-du-dépôt)
12. [Feuille de route](#12-feuille-de-route)

---

## 1. Contexte et motivation

La version 1 du système de veille reposait sur un ensemble d'outils SaaS propriétaires (Feedly, Talkwalker, Notion) présentant des limites structurelles incompatibles avec les thématiques étudiées :

| Limitation v1 | Impact |
|---|---|
| Données hébergées chez des tiers (Feedly, Notion) | Contradiction directe avec la thématique **Souveraineté** |
| Coût mensuel cumulé des licences SaaS | Non aligné avec les principes **FinOps** |
| Dépendance fournisseur (vendor lock-in) | Risque de continuité de service |
| Automatisation limitée par les tiers payants | Pipeline incomplet |

Le POC v2 explore un **stack 100% open source et auto-hébergé**, cohérent avec les trois thématiques de la veille, en ajoutant une fonctionnalité différenciante : la **production automatisée d'un podcast audio** via NotebookLM.

---

## 2. Problématique centrale

> *« Comment l'équation entre maîtrise des coûts (FinOps), urgence écologique (Green IT) et souveraineté des données reconfigure-t-elle les frontières entre Cloud, On-Premise et Hybride pour les années à venir ? »*

### Objectifs stratégiques

- **Anticiper** les mouvements de rapatriement cloud (*cloud repatriation*)
- **Évaluer** le TCO réel des infrastructures hybrides à horizon 3 ans
- **Suivre** l'évolution réglementaire européenne (RGPD, AI Act, Cyber Resilience Act)
- **Intégrer** les critères d'éco-responsabilité dans les décisions d'architecture

---

## 3. Stack technique v2

```
┌──────────────────────────────────────────────────────────────────────┐
│                         STACK TECHNIQUE v2                            │
├─────────────────┬──────────────────┬──────────────┬──────────────────┤
│    COLLECTE     │  AUTOMATISATION  │   STOCKAGE   │  COMMUNICATION   │
├─────────────────┼──────────────────┼──────────────┼──────────────────┤
│   Miniflux      │      n8n         │ Google Drive │      Slack       │
│  (self-hosted)  │  (self-hosted)   │  (structuré) │   (3 canaux)     │
│                 │                  │              │                  │
│  + RSS/Atom     │  + 3 workflows   │  + /01_Raw/  │  #veille-critique│
│  + API REST     │  + JS natif      │  + /02_Digest│  #veille-hebdo   │
│  + PostgreSQL   │  + Cron intégré  │  + /03_Podcast  #veille-podcast │
│                 │                  │  + /04_Reports              │   │
└─────────────────┴──────────────────┴──────────────┴──────────────────┘
                                                            │
                                        ┌───────────────────▼──────────┐
                                        │    PRODUCTION PODCAST         │
                                        │    NotebookLM (Google)        │
                                        │    Audio Overview (.m4a)      │
                                        └──────────────────────────────┘
```

### Justification des choix techniques

| Outil | Rôle | Justification |
|---|---|---|
| **Miniflux** | Agrégateur RSS | Open source, self-hosted, API REST complète, RGPD natif |
| **n8n** | Orchestration | Open source, self-hosted, JavaScript natif, 0€ |
| **Google Drive** | Stockage | 15 Go gratuits, API robuste, partage équipe |
| **Slack** | Communication | Webhooks entrants gratuits, intégrations riches |
| **NotebookLM** | Podcast audio | Audio Overview IA, gratuit, sources citées |

---

## 4. Architecture et infrastructure

Voir le schéma détaillé : [docs/poc_architecture.md](docs/poc_architecture.md)

### Infrastructure physique

```
┌──────────────────────────────────────────────────────────────┐
│              PROXMOX — Hyperviseur on-premise                 │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  VM : i2-infra-veille-g1 — Debian 12 (Bookworm)     │    │
│  │  IP : 172.16.89.44 · 2 vCPU · 2 Go RAM · 20 Go      │    │
│  │                                                       │    │
│  │  ┌──────────────┐    ┌──────────────┐               │    │
│  │  │ PostgreSQL 15│    │   Miniflux   │               │    │
│  │  │  Port 5432   │◄───│  Port 8080   │               │    │
│  │  └──────────────┘    └──────┬───────┘               │    │
│  │                             │                         │    │
│  │  ┌──────────────┐    ┌──────▼───────┐               │    │
│  │  │    Nginx     │    │     n8n      │               │    │
│  │  │  Port 80     │    │  Port 5678   │               │    │
│  │  └──────────────┘    └──────────────┘               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                               │
│  Accès : SSH tunnel depuis le poste client                   │
└──────────────────────────────────────────────────────────────┘
                          │
                          │ HTTPS sortant (VM → Internet)
                          ▼
┌──────────────────────────────────────────────────────────────┐
│                    SERVICES CLOUD (SaaS)                       │
│                                                               │
│  Google Drive          Slack              NotebookLM          │
│  /01_Raw/              #veille-critique   Notebook            │
│  /02_Digest/           #veille-hebdo      Veille EPSI 2026    │
│  /03_Podcast/          #veille-podcast                        │
│  /04_Reports/                                                  │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Workflows automatisés

Trois workflows n8n orchestrent le pipeline de veille.

### Workflow 1 — Collecte des articles (toutes les 2 heures)

```
Schedule Trigger (2h)
    → HTTP Request    — GET /v1/entries?status=unread (Miniflux API)
    → Split Out       — Séparer les entrées
    → Code (filtre)   — Filtrage par mots-clés thématiques
    → Code (markdown) — Construction du fichier .md
    → Google Drive    — Upload → /01_Raw/article_AAAA-MM-JJ_ID.md
    → HTTP Request    — PUT /v1/entries (marquer comme lu dans Miniflux)
    → HTTP Request    — POST Slack #veille-critique (articles critiques)
```

### Workflow 2 — Digest hebdomadaire (lundi 08h00)

```
Schedule Trigger (lundi 08h)
    → Code            — Calcul timestamp 7 jours, numéro de semaine
    → HTTP Request    — GET /v1/entries?after=TIMESTAMP (Miniflux API)
    → Code (digest)   — Agrégation par thématique + construction .md
    → Google Drive    — Upload → /02_Digest/digest_semaine_XX_AAAA-MM-JJ.md
    → HTTP Request    — POST Slack #veille-hebdo (résumé + action NotebookLM)
```

### Workflow 3 — Notification podcast (à la détection d'un fichier)

```
Google Drive Trigger  — Nouveau fichier dans /03_Podcast/
    → HTTP Request    — POST Slack #veille-podcast (lien d'écoute)
```

### Mots-clés de filtrage

| Thématique | Mots-clés (FR + EN) |
|---|---|
| **FinOps** | `finops`, `cloud cost`, `tco`, `cost optimization`, `capex`, `opex`, `cloud billing`, `coût du cloud`, `budget cloud` |
| **Green IT** | `green it`, `énergie`, `datacenter`, `empreinte carbone`, `sustainability`, `carbon`, `renewable energy`, `sobriété numérique` |
| **Souveraineté** | `sovereignty`, `souveraineté`, `gdpr`, `rgpd`, `cloud repatriation`, `ai act`, `données personnelles`, `conformité` |

---

## 6. Organisation Google Drive

```
📁 Veille_Techno_2026/
├── 📁 01_Raw/             ← Articles individuels (1 fichier = 1 article)
│   └── article_AAAA-MM-JJ_ID.md
├── 📁 02_Digest/          ← Synthèses hebdomadaires (1 fichier = 1 semaine)
│   └── digest_semaine_XX_AAAA-MM-JJ.md
├── 📁 03_Podcast/         ← Fichiers audio NotebookLM (.m4a)
│   └── podcast_semaine_XX.m4a
└── 📁 04_Reports/         ← Rapports mensuels (1 fichier = 1 mois)
    └── rapport_mensuel_MOIS_AAAA.md
```

### Structure d'un article (01_Raw)

```markdown
---
date: 2026-05-20
source: Next - Flux Complet
titre: "Titre de l'article"
url: https://...
tags: []
---

## Résumé

[Extrait automatique du flux RSS]

## Source originale

https://...
```

### Structure d'un digest (02_Digest)

```markdown
# Digest Veille Technologique — Semaine XX
**Période** : semaine du AAAA-MM-JJ
**Total articles** : N

## 💰 FinOps (N articles)
## 🌿 Green IT (N articles)
## 🔐 Souveraineté des données (N articles)
## 📌 Autres (N articles)
```

---

## 7. Communication Slack

| Canal | Usage | Déclencheur |
|---|---|---|
| `#veille-critique` | Alertes articles prioritaires | Workflow 1 — temps réel |
| `#veille-hebdo` | Digest de la semaine + action NotebookLM | Workflow 2 — lundi 08h |
| `#veille-podcast` | Lien vers le podcast audio | Workflow 3 — à la détection |

---

## 8. Production du podcast — NotebookLM

NotebookLM ne dispose pas d'API publique. La production du podcast est **semi-automatisée** :

| Étape | Responsable | Durée |
|---|---|---|
| Création du digest | n8n (automatique) | 0 min |
| Notification Slack | n8n (automatique) | 0 min |
| Ouverture NotebookLM + ajout source | Équipe (manuel) | 2 min |
| Génération Audio Overview | NotebookLM (automatique) | ~10 min |
| Téléchargement + dépôt dans /03_Podcast/ | Équipe (manuel) | 2 min |
| Notification Slack #veille-podcast | n8n (automatique) | 0 min |

**Total intervention humaine : ~4 minutes par semaine.**

### Paramétrage NotebookLM

1. Notebook dédié : `Veille EPSI 2026`
2. Sources permanentes : glossaires FinOps, Green IT, Souveraineté
3. Source hebdomadaire : `digest_semaine_XX.md` depuis Google Drive
4. Prompt Audio Overview :

```
Génère un podcast de veille technologique en français pour une équipe
d'étudiants en Master EPSI. Thèmes : FinOps, Green IT, Souveraineté
des données. Ton professionnel mais accessible.
Structure : introduction, 3 segments thématiques, tendance de la semaine.
```

---

## 9. Comparatif v1 vs v2

| Critère | Version 1 (SaaS) | Version 2 (POC Open Source) |
|---|---|---|
| **Collecte** | Feedly + Google Alerts | Miniflux self-hosted |
| **Automatisation** | Make.com + Zapier | **n8n self-hosted** |
| **Stockage** | Notion + Google Drive | Google Drive |
| **Communication** | Slack + Teams | Slack |
| **Coût mensuel** | ~80–150€ | **0€** (infra existante) |
| **Souveraineté données** | Partielle | **Totale** (VM on-premise) |
| **RGPD** | Dépend des CGU tiers | **Contrôle total** |
| **Podcast audio** | Non | **Oui** (NotebookLM) |
| **Complexité déploiement** | Faible | Moyenne |
| **Vendor lock-in** | Fort | **Nul** |

---

## 10. KPIs et métriques

| Métrique | Cible | Outil de mesure |
|---|---|---|
| Sources RSS actives | ≥ 10 flux | Miniflux Dashboard |
| Articles collectés / semaine | 50–100 | n8n logs |
| Articles catégorisés / semaine | ≥ 15 | Digest hebdomadaire |
| Digest généré / semaine | 1 | Google Drive /02_Digest/ |
| Podcast produit / semaine | 1 | Google Drive /03_Podcast/ |
| Alertes Slack critiques / semaine | 2–10 | Slack analytics |
| Disponibilité Miniflux | > 99% | systemd status |
| Coût infrastructure | 0€ | — |

---

## 11. Structure du dépôt

```
veille_techno_2026/
├── README.md                      ← Version 1 (Stack SaaS)
├── README_v2_POC.md               ← Ce fichier (Version 2 POC)
│
├── config/
│   ├── miniflux_config.md         ← Guide déploiement Miniflux (Debian natif)
│   ├── n8n_workflows.md           ← Documentation des 3 workflows n8n
│   └── feedly_opml.xml            ← Export OPML des flux RSS
│
├── docs/
│   ├── poc_architecture.md        ← Schémas d'infrastructure (Mermaid)
│   ├── sources.md                 ← Cartographie des sources RSS
│   ├── kpi_metrics.md             ← Indicateurs de performance
│   ├── notebooklm_podcast.md      ← Guide production podcast
│   └── problematique.md           ← Contexte stratégique
│
├── templates/
│   ├── monthly_report.md          ← Template rapport mensuel
│   └── decision_matrix.md         ← Matrice Cloud vs On-Prem
│
└── reports/
    └── 2026/
        └── 03-mars-report.md
```

---

## 12. Feuille de route

### ✅ Phase 1 — Infrastructure (réalisée)

- [x] VM Proxmox Debian 12 provisionnée
- [x] Miniflux déployé (binaire natif, service systemd)
- [x] PostgreSQL 15 configuré
- [x] Nginx reverse proxy configuré
- [x] n8n déployé (npm, service systemd)
- [x] Flux RSS configurés (10+ sources)

### ✅ Phase 2 — Automatisation (réalisée)

- [x] Workflow 1 : collecte toutes les 2h → Google Drive /01_Raw/
- [x] Workflow 2 : digest hebdomadaire → Google Drive /02_Digest/
- [x] Workflow 3 : notification podcast → Slack #veille-podcast
- [x] Filtrage par mots-clés (FR + EN)
- [x] Notifications Slack (3 canaux)
- [x] Marquage articles lus dans Miniflux

### ✅ Phase 3 — Podcast (réalisée)

- [x] Notebook NotebookLM créé
- [x] Premier podcast généré depuis digest_semaine_21
- [x] Fichier .m4a déposé dans /03_Podcast/
- [x] Notification Slack automatique

### 🔧 Phase 4 — Optimisation (en cours)

- [ ] Enrichissement des flux RSS (sources spécialisées FinOps)
- [ ] Activation "Fetch original content" sur tous les flux
- [ ] Rapport mensuel automatisé (04_Reports)
- [ ] Sauvegarde automatique PostgreSQL

---

*Documentation rédigée pour le projet de veille technologique — Master EPSI 2025-2026*  
*Infrastructure : VM Proxmox (Debian 12) · Miniflux · n8n · Google Drive · Slack · NotebookLM*
