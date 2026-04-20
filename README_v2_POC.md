# Système de Veille Technologique — Version POC (v2)
## Stack Open Source : MiniFlux · Make.com · Google Drive · Slack · NotebookLM

> **Version** : 2.0 — Proof of Concept  
> **Basé sur** : [README.md](README.md) (Version 1 — Stack SaaS)  
> **Thématiques** : FinOps · Green IT · Souveraineté des données  
> **Équipe** : Serge WEMBE II-ESSOUMBA · Cheik LAWANI · Javier LADINO

---

## Sommaire

1. [Contexte et motivation du POC](#1-contexte-et-motivation-du-poc)
2. [Problématique centrale](#2-problématique-centrale)
3. [Stack technique v2](#3-stack-technique-v2)
4. [Architecture et pipeline de données](#4-architecture-et-pipeline-de-données)
5. [Déploiement de MiniFlux](#5-déploiement-de-miniflux)
6. [Configuration Make.com (scénario v2)](#6-configuration-makecom-scénario-v2)
7. [Organisation Google Drive](#7-organisation-google-drive)
8. [Automatisation Slack](#8-automatisation-slack)
9. [Production du podcast avec NotebookLM](#9-production-du-podcast-avec-notebooklm)
10. [Workflow opérationnel](#10-workflow-opérationnel)
11. [Comparatif v1 vs v2](#11-comparatif-v1-vs-v2)
12. [KPIs et métriques](#12-kpis-et-métriques)
13. [Structure du dépôt](#13-structure-du-dépôt)
14. [Feuille de route](#14-feuille-de-route)

---

## 1. Contexte et motivation du POC

La version 1 du système de veille repose sur un ensemble d'outils SaaS propriétaires (Feedly, Talkwalker, Notion) qui présentent plusieurs limites dans un contexte de souveraineté des données et de maîtrise des coûts :

| Limitation v1 | Impact |
|---|---|
| Données hébergées chez des tiers (Feedly, Notion) | Contradiction avec la thématique souveraineté |
| Coût mensuel cumulé des licences SaaS | Non aligné avec les principes FinOps |
| Dépendance fournisseur (vendor lock-in) | Risque de continuité de service |
| Intégrations API limitées en tier gratuit | Automatisation incomplète |

Ce POC explore un **stack alternatif centré sur l'open source et la maîtrise des données**, tout en ajoutant une fonctionnalité différenciante : **la production automatisée d'un podcast audio** via NotebookLM.

---

## 2. Problématique centrale

> *"Comment l'équation entre maîtrise des coûts (FinOps), urgence écologique (Green IT) et souveraineté des données reconfigure-t-elle les frontières entre Cloud, On-Premise et Hybride pour les années à venir ?"*

### Objectifs stratégiques

- **Anticiper** les mouvements de rapatriement cloud (cloud repatriation)
- **Évaluer** le TCO réel des infrastructures hybrides à horizon 3 ans
- **Suivre** l'évolution réglementaire européenne (RGPD, Cyber Resilience Act, AI Act)
- **Intégrer** les critères d'éco-responsabilité dans les décisions d'architecture

---

## 3. Stack technique v2

```
┌─────────────────────────────────────────────────────────────────┐
│                        STACK TECHNIQUE v2                        │
├────────────────┬────────────────┬───────────────┬───────────────┤
│   COLLECTE     │ AUTOMATISATION │   STOCKAGE    │ COMMUNICATION │
│                │                │   & ANALYSE   │               │
├────────────────┼────────────────┼───────────────┼───────────────┤
│  MiniFlux      │   Make.com     │  Google Drive │     Slack     │
│  (self-hosted) │  (scénario v2) │  (structuré)  │  (3 canaux)   │
│                │                │               │               │
│  + RSS/Atom    │  + Webhooks    │  + Sheets     │  + Webhooks   │
│  + API REST    │  + Filtres     │  + Docs       │  + Bots       │
│  + Docker      │  + Routage     │  + Drive API  │               │
└────────────────┴────────────────┴───────────────┴───────────────┘
                                                        │
                                    ┌───────────────────▼──────────┐
                                    │  PRODUCTION PODCAST           │
                                    │  NotebookLM (Google)          │
                                    │  Audio Overview automatisé    │
                                    └──────────────────────────────┘
```

### Justification des choix techniques

| Outil | Rôle | Justification |
|---|---|---|
| **MiniFlux** | Agrégateur RSS | Open source, self-hosted, API-first, RGPD-compatible |
| **Make.com** | Orchestration | Connexions natives Google Drive + Slack, scénarios visuels |
| **Google Drive** | Stockage | Gratuit jusqu'à 15 Go, API robuste, partage équipe |
| **Slack** | Communication | Webhooks entrants gratuits, intégrations riches |
| **NotebookLM** | Podcast audio | IA de synthèse + Audio Overview, gratuit, sources citées |

---

## 4. Architecture et pipeline de données

Voir le schéma détaillé : [docs/poc_architecture.md](docs/poc_architecture.md)

### Vue d'ensemble du flux

```
Sources RSS                    MiniFlux              Make.com
─────────────                 ──────────            ──────────
FinOps feeds ──────────────►  Self-hosted  ──────►  Polling API
GreenIT feeds ─────────────►  Docker       │        Filtre mots-clés
Souveraineté feeds ─────────►  :8080        │        Routage intelligent
                                            │
                    ┌───────────────────────┘
                    │
                    ▼
         ┌──────────────────────────────────────────┐
         │           MAKE.COM — ROUTEUR              │
         │                                          │
         │  [Critique] ──────► Slack #veille-critique│
         │  [Standard] ──────► Google Drive /Raw     │
         │  [Hebdo]    ──────► Google Drive /Digest  │
         │  [Digest]   ──────► NotebookLM sources    │
         └──────────────────────────────────────────┘
                    │                    │
                    ▼                    ▼
              Slack channels      Google Drive
              #veille-critique    /Veille_2026/
              #veille-hebdo         Raw/
              #veille-podcast       Digest/
                                    Podcast/
                    │                    │
                    └────────────────────┘
                                │
                                ▼
                          NotebookLM
                    (Upload digest hebdo)
                    Audio Overview généré
                                │
                                ▼
                    Google Drive /Podcast/
                    podcast_semaine_XX.mp3
                                │
                                ▼
                    Slack #veille-podcast
                    (lien de partage)
```

---

## 5. Déploiement de MiniFlux

Voir le guide complet : [config/miniflux_config.md](config/miniflux_config.md)

### Prérequis

- Docker Desktop (Mac/Windows/Linux) ou VPS (OVHcloud, Scaleway)
- Accès terminal / SSH
- Port 8080 disponible (ou configurable)

### Démarrage rapide avec Docker Compose

```yaml
# docker-compose.yml
version: '3'
services:
  miniflux:
    image: miniflux/miniflux:latest
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://miniflux:secret@db/miniflux?sslmode=disable
      - RUN_MIGRATIONS=1
      - CREATE_ADMIN=1
      - ADMIN_USERNAME=veille_admin
      - ADMIN_PASSWORD=VeilleEPSI2026!
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=miniflux
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=miniflux
    volumes:
      - miniflux_db:/var/lib/postgresql/data

volumes:
  miniflux_db:
```

```bash
# Lancement
docker compose up -d

# Accès interface web
open http://localhost:8080

# Vérification logs
docker compose logs -f miniflux
```

### Flux RSS à configurer

```
FINOPS :
  https://feeds.feedburner.com/finops-foundation
  https://www.finops.org/feed/
  https://cloudoptimizer.net/feed/

GREEN IT :
  https://www.greenit.fr/feed/
  https://www.wwf.fr/feed
  https://www.iea.org/rss/news.xml

SOUVERAINETÉ :
  https://www.silicon.fr/feed
  https://www.lemondeinformatique.fr/flux-rss/thematique/infrastructure/rss.xml
  https://www.nextinpact.com/rss/articles.xml
  https://www.cnil.fr/fr/rss.xml
```

---

## 6. Configuration Make.com (scénario v2)

Voir le blueprint JSON : [config/make_scenario_v2.json](config/make_scenario_v2.json)

### Scénario principal — Collecte & Distribution

**Étape 1 — Déclencheur (Trigger)**
- Module : `HTTP - Make a request`
- URL : `http://[MINIFLUX_HOST]:8080/v1/entries?status=unread&limit=50`
- Headers : `X-Auth-Token: [MINIFLUX_API_KEY]`
- Fréquence : toutes les 2 heures

**Étape 2 — Filtrage par mots-clés**
```
Condition OR :
  titre CONTIENT "FinOps"
  titre CONTIENT "Green IT" OU "GreenIT"
  titre CONTIENT "souveraineté" OU "sovereignty"
  titre CONTIENT "cloud repatriation" OU "rapatriement"
  titre CONTIENT "TCO" OU "CAPEX" OU "OPEX"
  titre CONTIENT "RGPD" OU "Cloud Act" OU "AI Act"
```

**Étape 3 — Routeur**

| Branche | Condition | Action |
|---|---|---|
| Critique | Score pertinence > 8 | Post Slack `#veille-critique` |
| Standard | Tous articles filtrés | Créer fichier Google Drive `/Raw/` |
| Hebdomadaire | Chaque lundi 08h00 | Générer digest → Google Drive `/Digest/` |

**Étape 4 — Actions parallèles**
- **Google Drive** : Créer document `{titre}_{date}.md` dans `/Veille_2026/Raw/`
- **Slack** : Envoyer message formaté avec titre, résumé, URL, tags thématiques
- **Google Sheets** : Ajouter ligne dans le tableau de suivi des articles

---

## 7. Organisation Google Drive

```
📁 Google Drive
└── 📁 Veille_Techno_2026/
    ├── 📁 00_Config/
    │   ├── make_scenario_v2.json
    │   └── miniflux_feeds.opml
    ├── 📁 01_Raw/
    │   └── 📁 2026/
    │       ├── 📁 04-avril/
    │       │   ├── article_finops_20260418.md
    │       │   └── article_greenit_20260417.md
    │       └── 📁 05-mai/
    ├── 📁 02_Digest/
    │   └── 📁 2026/
    │       ├── digest_semaine_16.md
    │       └── digest_semaine_17.md
    ├── 📁 03_Podcast/
    │   ├── podcast_semaine_16.mp3
    │   └── podcast_semaine_17.mp3
    ├── 📁 04_Reports/
    │   └── rapport_mensuel_avril_2026.md
    └── 📄 suivi_articles.gsheet   ← Tableau de bord équipe
```

### Structure d'un fichier article (Raw)

```markdown
---
date: 2026-04-18
source: Le Monde Informatique
titre: "Le cloud repatriation s'accélère en Europe"
url: https://...
tags: [FinOps, Cloud Repatriation, Souveraineté]
score_pertinence: 9/10
thematique: Souveraineté
---

## Résumé

[Résumé automatique Make.com / extrait RSS]

## Mots-clés identifiés

FinOps, TCO, cloud repatriation, RGPD

## Source originale

[URL complète]
```

---

## 8. Automatisation Slack

### Canaux requis

| Canal | Usage | Fréquence |
|---|---|---|
| `#veille-critique` | Alertes prioritaires (score ≥ 8) | Temps réel |
| `#veille-hebdo` | Digest de la semaine | Lundi 08h00 |
| `#veille-podcast` | Lien vers le podcast audio | Vendredi 17h00 |

### Format message Slack — Alerte critique

```
🚨 *VEILLE CRITIQUE — FinOps*

*[Titre de l'article]*
📰 Source : Le Monde Informatique
🏷️ Tags : #FinOps #TCO #CloudRepatriation
📅 Date : 18 avril 2026

> Résumé : [Extrait automatique du flux RSS]

🔗 Lire l'article : https://...
📁 Archivé dans : Google Drive /Raw/04-avril/
```

### Format message Slack — Digest hebdomadaire

```
📊 *DIGEST VEILLE — Semaine 16 (14-18 avril 2026)*

*Top 5 articles de la semaine :*

1. 📈 [FinOps] Titre article 1 — source
2. 🌿 [Green IT] Titre article 2 — source
3. 🔐 [Souveraineté] Titre article 3 — source
4. ☁️ [Cloud] Titre article 4 — source
5. 💡 [Hybride] Titre article 5 — source

📁 Digest complet : [lien Google Drive]
🎙️ Podcast de la semaine : [lien audio]

_Généré automatiquement par le système de veille EPSI_
```

---

## 9. Production du podcast avec NotebookLM

Voir le guide détaillé : [docs/notebooklm_podcast.md](docs/notebooklm_podcast.md)

### Principe

**NotebookLM** (Google) propose une fonctionnalité **Audio Overview** qui génère automatiquement un podcast de style conversation entre deux présentateurs IA à partir de sources textuelles fournies. Ce podcast synthétise les points clés des sources avec un ton naturel et pédagogique.

### Workflow de production (hebdomadaire)

```
[Vendredi 16h00] Make.com déclenche la génération du digest hebdo
        │
        ▼
Google Drive : digest_semaine_XX.md créé
        │
        ▼
[Vendredi 16h15] Make.com upload le digest dans NotebookLM
        │         via Google Drive API / partage de lien
        ▼
NotebookLM : Audio Overview généré (~10-15 minutes de traitement)
        │
        ▼
Téléchargement du fichier MP3 généré
        │
        ▼
Google Drive : /03_Podcast/podcast_semaine_XX.mp3 sauvegardé
        │
        ▼
[Vendredi 17h00] Slack #veille-podcast : lien partagé
```

### Contenu type d'un podcast (15-20 min)

```
Segment 1 — Introduction (2 min)
  "Cette semaine dans la veille EPSI, nous avons relevé X articles..."

Segment 2 — FinOps (5 min)
  Synthèse des actualités TCO, cloud costs, repatriation

Segment 3 — Green IT (4 min)
  Bilan carbone numérique, nouvelles réglementations

Segment 4 — Souveraineté des données (4 min)
  RGPD, Cloud Act, directives européennes

Segment 5 — Tendance de la semaine (3 min)
  Article le plus impactant + recommandation équipe
```

### Paramétrage NotebookLM

1. Créer un **notebook dédié** : `Veille EPSI 2026`
2. Configurer les sources permanentes :
   - Glossaire FinOps (uploadé une fois)
   - Glossaire Green IT
   - Contexte du projet (problematique.md)
3. Chaque semaine : ajouter le digest comme **source temporaire**
4. Lancer **Audio Overview** avec le prompt de contexte :

```
Génère un podcast de veille technologique en français pour une équipe 
d'étudiants en Master EPSI. Les thèmes sont FinOps, Green IT et 
Souveraineté des données. Ton professionnel mais accessible. 
Durée cible : 15 minutes. Structure : intro, 3 segments thématiques, 
tendance de la semaine.
```

### Limites actuelles et évolutions

| Aspect | Situation actuelle | Évolution prévue |
|---|---|---|
| Déclenchement | Manuel (UI NotebookLM) | API NotebookLM (bêta annoncée) |
| Langue | Principalement anglais | Support français en amélioration |
| Format | MP3 téléchargeable | Lien de partage direct |
| Durée | 15-20 min générées | Configurable via prompt |

---

## 10. Workflow opérationnel

### Cadence

| Fréquence | Action | Responsable | Outil |
|---|---|---|---|
| Temps réel | Alertes critiques | Make.com | → Slack #veille-critique |
| Toutes les 2h | Collecte MiniFlux | Make.com | → Google Drive /Raw/ |
| Lundi 08h00 | Digest hebdomadaire | Make.com | → Google Drive /Digest/ |
| Vendredi 16h00 | Génération podcast | Make.com + NotebookLM | → Google Drive /Podcast/ |
| Vendredi 17h00 | Partage podcast | Make.com | → Slack #veille-podcast |
| 1er du mois | Rapport mensuel | Équipe | → Google Drive /Reports/ |

### Routine d'équipe

```bash
# 1. Consultation des alertes critiques
→ Slack #veille-critique (temps réel)

# 2. Revue du digest hebdomadaire
→ Slack #veille-hebdo (lundi matin)
→ Google Drive /Digest/ pour le document complet

# 3. Écoute du podcast
→ Slack #veille-podcast (vendredi soir)
→ Google Drive /Podcast/ pour téléchargement

# 4. Contribution au rapport mensuel
→ Google Drive /Reports/ (début de mois)
→ PR GitHub sur le dépôt veille_techno_2026
```

---

## 11. Comparatif v1 vs v2

| Critère | Version 1 (SaaS) | Version 2 (POC Open Source) |
|---|---|---|
| **Collecte** | Feedly + Google Alerts + Talkwalker | MiniFlux (self-hosted) |
| **Automatisation** | Make.com + Zapier | Make.com uniquement |
| **Stockage** | Notion + Google Drive | Google Drive uniquement |
| **Communication** | Slack + Microsoft Teams | Slack uniquement |
| **Coût mensuel estimé** | ~80-150€ | ~5-20€ (hébergement VPS) |
| **Souveraineté données** | Partielle (données chez Feedly/Notion) | Améliorée (MiniFlux self-hosted) |
| **RGPD** | Dépend des CGU tiers | Contrôle total sur MiniFlux |
| **Podcast audio** | Non | Oui (NotebookLM) |
| **Complexité déploiement** | Faible (tout SaaS) | Moyenne (Docker requis) |
| **Maintenance** | Aucune | Mise à jour Docker périodique |
| **Scalabilité** | Illimitée (SaaS) | Limitée au VPS |

---

## 12. KPIs et métriques

| Métrique | Cible | Mesure |
|---|---|---|
| Sources surveillées | ≥ 50 flux RSS | MiniFlux Dashboard |
| Articles collectés/semaine | 20-30 | Google Sheets suivi |
| Articles analysés/semaine | 5-10 | Google Sheets suivi |
| Alertes critiques/semaine | 2-5 | Slack analytics |
| Podcast généré/semaine | 1 | Google Drive /Podcast/ |
| Temps de réaction (alerte critique) | < 2h | Make.com logs |
| Disponibilité MiniFlux | > 99% | Monitoring Docker |
| Coût infrastructure | < 20€/mois | Facture VPS |

---

## 13. Structure du dépôt

```
veille_techno_2026/
├── README.md                    ← Version 1 (Stack SaaS)
├── README_v2_POC.md             ← Ce fichier (Version 2 POC)
│
├── config/
│   ├── alerts_setup.md          ← Configuration Google Alerts (v1)
│   ├── feedly_opml.xml          ← Export Feedly (v1)
│   ├── make_scenario.json       ← Blueprint Make.com v1
│   ├── make_scenario_v2.json    ← Blueprint Make.com v2 (POC)
│   └── miniflux_config.md       ← Guide déploiement MiniFlux
│
├── docs/
│   ├── problematique.md         ← Contexte stratégique
│   ├── sources.md               ← Cartographie des sources
│   ├── kpi_metrics.md           ← Indicateurs de performance
│   ├── poc_architecture.md      ← Schéma infrastructure v2
│   └── notebooklm_podcast.md   ← Guide production podcast
│
├── templates/
│   ├── monthly_report.md        ← Template rapport mensuel
│   └── decision_matrix.md       ← Matrice Cloud vs On-Prem
│
└── reports/
    └── 2026/
        └── 03-mars-report.md
```

---

## 14. Feuille de route

### Phase 1 — Déploiement infrastructure (Semaine 1-2)
- [ ] Déployer MiniFlux sur VPS ou Docker local
- [ ] Configurer les flux RSS (≥ 30 sources)
- [ ] Obtenir la clé API MiniFlux
- [ ] Créer la structure Google Drive

### Phase 2 — Automatisation Make.com (Semaine 2-3)
- [ ] Importer le scénario `make_scenario_v2.json`
- [ ] Configurer les connexions Google Drive + Slack
- [ ] Tester le pipeline de collecte end-to-end
- [ ] Valider les filtres de mots-clés

### Phase 3 — Podcast NotebookLM (Semaine 3-4)
- [ ] Créer le notebook NotebookLM dédié
- [ ] Uploader les sources contextuelles permanentes
- [ ] Tester la génération Audio Overview
- [ ] Automatiser la distribution Slack

### Phase 4 — Validation et rapport (Semaine 4)
- [ ] Mesurer les KPIs sur 2 semaines
- [ ] Rédiger le rapport comparatif v1 vs v2
- [ ] Présentation équipe + recommandations

---

*Documentation générée pour le projet de veille technologique — Master EPSI 2025-2026*  
*Stack : MiniFlux · Make.com · Google Drive · Slack · NotebookLM*
