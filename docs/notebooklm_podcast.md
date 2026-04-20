# Production de podcast avec NotebookLM — Guide opérationnel

> **Fonctionnalité** : Audio Overview automatisé  
> **Outil** : Google NotebookLM  
> **Objectif** : Générer un podcast hebdomadaire de synthèse de la veille technologique  
> **Langue cible** : Français  

---

## Qu'est-ce que NotebookLM Audio Overview ?

**NotebookLM** est un outil d'IA de Google (basé sur Gemini) qui permet d'analyser des sources textuelles et de générer des contenus synthétiques. La fonctionnalité **Audio Overview** crée automatiquement un podcast en format conversation entre deux présentateurs IA à partir des sources fournies.

### Caractéristiques

| Aspect | Détail |
|---|---|
| **Format** | Conversation entre 2 présentateurs IA |
| **Durée** | 10-25 minutes (selon volume des sources) |
| **Qualité** | Naturelle, fluide, ton journalistique |
| **Langue** | Multilingue (français pris en charge) |
| **Sources supportées** | PDF, Google Docs, texte brut, URLs, YouTube |
| **Export** | Téléchargement MP3 direct |
| **Coût** | Gratuit (compte Google requis) |

---

## Configuration initiale du notebook

### Étape 1 — Créer le notebook dédié

1. Accéder à **notebooklm.google.com**
2. Cliquer sur **"Nouveau notebook"**
3. Nommer : `Veille Technologique EPSI 2026`
4. Icône : Choisir une icône représentative

### Étape 2 — Ajouter les sources contextuelles permanentes

Ces sources sont ajoutées une seule fois et restent dans le notebook pour donner le contexte permanent à l'IA.

#### Source 1 — Contexte du projet

Créer un document Google Docs `contexte_veille_epsi.md` avec le contenu :

```markdown
# Contexte Projet — Veille Technologique EPSI 2026

## Équipe
- Serge WEMBE II-ESSOUMBA
- Cheik LAWANI  
- Javier LADINO

## Problématique
Comment l'équation entre maîtrise des coûts (FinOps), urgence écologique (Green IT) 
et souveraineté des données reconfigure-t-elle les frontières entre Cloud, On-Premise 
et Hybride pour les années à venir ?

## Thématiques surveillées

### FinOps
Optimisation des coûts cloud, TCO (Total Cost of Ownership), CAPEX vs OPEX, 
cloud repatriation (rapatriement vers on-premise), rightsizing.

### Green IT
Empreinte carbone du numérique, PUE des datacenters, numérique responsable, 
efficacité énergétique, réglementations environnementales.

### Souveraineté des données
RGPD, Cloud Act américain, AI Act européen, données personnelles, 
localisation des données, fournisseurs cloud européens.

## Contexte 2026
- Accélération du cloud repatriation en Europe
- Directives européennes de souveraineté numérique
- Pression réglementaire croissante sur les hyperscalers américains
- Maturation de l'edge computing
```

#### Source 2 — Glossaire technique

Créer un document `glossaire_veille.md` avec les définitions clés :

```markdown
# Glossaire Veille Technologique EPSI 2026

**FinOps** : Pratique financière appliquée au cloud, visant à maximiser la valeur 
métier tout en contrôlant les coûts d'infrastructure.

**Green IT** : Informatique verte, ensemble des pratiques visant à réduire l'impact 
environnemental des technologies numériques.

**Cloud Repatriation** : Mouvement de migration inverse, depuis le cloud public 
vers des infrastructures on-premise ou des clouds privés.

**TCO (Total Cost of Ownership)** : Coût total de possession, incluant CAPEX, OPEX, 
maintenance, formation et coûts cachés.

**CAPEX** : Capital Expenditure, dépenses d'investissement (achat serveurs, licences).

**OPEX** : Operational Expenditure, dépenses opérationnelles (abonnements, consommation).

**Souveraineté numérique** : Capacité d'un état ou d'une organisation à contrôler 
ses données et ses systèmes informatiques.

**RGPD** : Règlement Général sur la Protection des Données, réglementation européenne 
sur la protection des données personnelles.

**Cloud Act** : Loi américaine permettant aux autorités américaines d'accéder aux 
données stockées par des entreprises américaines, même en Europe.

**PUE (Power Usage Effectiveness)** : Ratio mesurant l'efficacité énergétique 
d'un datacenter.

**Edge Computing** : Traitement des données au plus près de la source (bordure du réseau), 
réduisant la latence et la dépendance au cloud central.
```

### Étape 3 — Configurer le prompt système du notebook

Dans NotebookLM, utiliser la fonctionnalité **"Instructions pour le notebook"** :

```
Tu es un assistant spécialisé en veille technologique pour une équipe d'étudiants 
en Master EPSI (École Supérieure de l'Informatique). 

Contexte : Nous surveillons les thématiques FinOps, Green IT et Souveraineté des 
données pour analyser l'évolution des infrastructures Cloud, On-Premise et Hybrides.

Pour l'Audio Overview hebdomadaire :
- Adopte un ton professionnel mais accessible, adapté à des étudiants en Master
- Privilégie le français
- Structure : introduction (2 min), FinOps (4 min), Green IT (4 min), 
  Souveraineté (4 min), tendance de la semaine (3 min)
- Cite les sources et les experts mentionnés dans les articles
- Termine par une recommandation concrète pour l'équipe
- Durée cible : 15-20 minutes
```

---

## Workflow hebdomadaire de production

### Vue d'ensemble

```
Vendredi 16h00 ──► Make.com génère le digest de la semaine
        │
        ▼
Digest semaine XX sauvegardé dans Google Drive /Digest/
        │
        ▼
Vendredi 16h15 ──► Upload manuel du digest dans NotebookLM
        │            (ou automatisé via Google Drive API)
        ▼
NotebookLM traite les sources (2-5 minutes)
        │
        ▼
Vendredi 16h20 ──► Lancer "Audio Overview" dans NotebookLM
        │
        ▼
Génération du podcast (5-15 minutes de traitement)
        │
        ▼
Vendredi 16h35-17h00 ──► Télécharger le MP3 généré
        │
        ▼
Uploader dans Google Drive /Podcast/podcast_semaine_XX.mp3
        │
        ▼
Vendredi 17h00 ──► Make.com envoie le lien Slack #veille-podcast
```

### Instructions détaillées

#### Ajout du digest comme source

1. Ouvrir le notebook `Veille Technologique EPSI 2026`
2. Cliquer sur **"+ Ajouter une source"**
3. Choisir **"Google Drive"** et sélectionner `digest_semaine_XX.md`
4. Attendre le traitement (indicateur de chargement)
5. Vérifier que la source apparaît dans le panneau sources

#### Génération de l'Audio Overview

1. Dans la section **"Studio"** (panneau droit)
2. Cliquer sur **"Audio Overview"**
3. Cliquer sur **"Personnaliser"** et saisir le prompt :

```
Génère un podcast de synthèse de veille technologique en FRANÇAIS.
Thèmes de la semaine : FinOps, Green IT, Souveraineté des données.
Audience : étudiants Master EPSI, niveau expert technique.
Structure souhaitée :
1. Intro : résumé de la semaine en 2 minutes
2. FinOps : actualités coûts cloud et repatriation (4 min)
3. Green IT : bilan carbone numérique et réglementations (4 min)  
4. Souveraineté : RGPD, Cloud Act, directives européennes (4 min)
5. Tendance de la semaine : article le plus impactant (3 min)
6. Conclusion : recommandation pour notre équipe de veille (2 min)
Ton : professionnel, dynamique, citations des experts mentionnés.
```

4. Cliquer sur **"Générer"**
5. Patienter 5-15 minutes (selon la longueur des sources)

#### Export et distribution

```bash
# 1. Télécharger le MP3 depuis NotebookLM
→ Cliquer sur le bouton de téléchargement audio
→ Renommer : podcast_semaine_XX_YYYY.mp3

# 2. Uploader dans Google Drive
→ Google Drive → /Veille_Techno_2026/03_Podcast/
→ Glisser-déposer le fichier MP3

# 3. Obtenir le lien de partage
→ Clic droit → Partager → Lien de partage
→ Accès : "Tous les utilisateurs avec le lien peuvent consulter"
→ Copier le lien

# 4. Partager sur Slack
→ Canal : #veille-podcast
→ Coller le lien + message descriptif
→ Épingler le message dans le canal
```

---

## Format du podcast généré

### Structure type (15-20 minutes)

```
[0:00 - 2:00] INTRODUCTION
  Présentateur A : "Bonjour et bienvenue dans le podcast de veille technologique 
  de l'équipe EPSI. Je suis [voix IA 1]."
  Présentateur B : "Et moi [voix IA 2]. Cette semaine, du 14 au 18 avril 2026, 
  nous avons recensé [X] articles sur les thèmes FinOps, Green IT et 
  souveraineté des données."
  [Présentation des 3 sujets phares de la semaine]

[2:00 - 6:00] SEGMENT FINOPS
  Actualités : optimisation des coûts cloud, nouveaux outils TCO, 
  tendances cloud repatriation
  Citation d'experts mentionnés dans les articles
  Impact pour les DSI et architectes cloud

[6:00 - 10:00] SEGMENT GREEN IT
  Rapport sur l'empreinte carbone du numérique
  Nouvelles certifications et labels environnementaux
  Initiatives des grands acteurs (GAFAM, OVHcloud, Scaleway)

[10:00 - 14:00] SEGMENT SOUVERAINETÉ DES DONNÉES
  Évolutions réglementaires RGPD, AI Act, Cyber Resilience Act
  Positionnement des cloud providers européens
  Cas pratiques souveraineté vs performance

[14:00 - 17:00] TENDANCE DE LA SEMAINE
  L'article le plus impactant analysé en profondeur
  Discussion sur les implications à 6-12 mois

[17:00 - 19:00] CONCLUSION
  Recommandation concrète pour l'équipe de veille
  Preview des sujets à surveiller la semaine suivante
  Appel à contribution de l'équipe
```

---

## Automatisation avancée (roadmap)

### Phase actuelle — Workflow semi-automatique

```
Make.com → Google Drive (digest)
Humain → NotebookLM (ajout source + génération)
Humain → Google Drive (upload MP3)
Make.com → Slack (notification)
```

### Phase future — Workflow entièrement automatique

L'API NotebookLM est en développement actif chez Google. L'automatisation complète sera possible lorsque les endpoints suivants seront disponibles :

```
POST /notebooks/{id}/sources     ← Ajouter une source
POST /notebooks/{id}/audio       ← Déclencher Audio Overview
GET  /notebooks/{id}/audio/status ← Vérifier l'avancement
GET  /notebooks/{id}/audio/download ← Télécharger le MP3
```

**Estimation disponibilité** : fin 2026 (selon roadmap Google)

#### Blueprint Make.com — Automatisation complète (future)

```json
{
  "id": "podcast_automation_v3",
  "trigger": "Vendredi 16h00",
  "steps": [
    "Lire digest_semaine.md depuis Google Drive",
    "POST NotebookLM API → ajouter source",
    "POST NotebookLM API → lancer Audio Overview",
    "Polling statut toutes les 2 minutes",
    "GET NotebookLM API → télécharger MP3",
    "PUT Google Drive → /Podcast/ sauvegarder",
    "POST Slack → #veille-podcast partager lien"
  ]
}
```

---

## Métriques et qualité

### KPIs du podcast

| Métrique | Cible | Méthode de mesure |
|---|---|---|
| Podcasts produits/mois | 4 (1/semaine) | Comptage dossier /Podcast/ |
| Durée moyenne | 15-20 min | Métadonnées fichier MP3 |
| Délai production | < 2h (vendredi) | Horodatage fichier |
| Couverture thématiques | 3/3 par épisode | Review manuelle |
| Mentions équipe dans Slack | ≥ 3 réactions | Slack analytics |

### Checklist qualité avant distribution

```
□ Vérifier que les 3 thématiques sont couvertes
□ Confirmer la durée (15-20 min)
□ Écouter les 2 premières minutes (intro correcte ?)
□ Vérifier que le fichier MP3 est accessible sur Google Drive
□ Lien de partage fonctionnel (accès "avec le lien")
□ Message Slack bien formaté avec contexte de la semaine
□ Fichier MP3 épinglé dans #veille-podcast
```

---

## Limites et points d'attention

| Limitation | Impact | Mitigation |
|---|---|---|
| Langue française | Quality variable vs anglais | Prompt explicite en français |
| Sources limitées (50 max) | Digest doit être synthétique | Sélectionner top 10 articles |
| Pas d'API publique | Workflow manuel partiellement | Semi-automatisation acceptable |
| Noms propres | Parfois mal prononcés | Accepter ou corriger dans digest |
| Citations exactes | IA paraphrase, ne cite pas verbatim | Vérifier avant distribution critique |
| Durée non contrôlable précisément | Peut varier 10-25 min | Ajuster volume des sources |

---

## Exemples de prompts efficaces

### Prompt FinOps focalisé

```
Génère un podcast de 10 minutes en français focalisé exclusivement sur FinOps 
et la gestion des coûts cloud. Audience : DSI et architectes cloud en Europe.
Inclure : tendances cloud repatriation, outils de cost optimization, 
benchmarks TCO publiccloud vs on-premise.
```

### Prompt Green IT urgence

```
Génère un podcast de 12 minutes en français sur l'urgence du Green IT en 2026.
Contexte réglementaire européen, directives carbones, certifications datacenter.
Ton : alerte et pragmatique. Inclure des chiffres concrets d'impact.
```

### Prompt souveraineté actualité

```
Génère un épisode de 15 minutes sur l'actualité de la souveraineté numérique 
en Europe en 2026. RGPD, AI Act, alternatives aux GAFAM (OVHcloud, Scaleway, 
Hetzner). Analyser les implications pour les entreprises françaises.
```

---

*Guide NotebookLM — Projet veille technologique EPSI 2025-2026*  
*Équipe : Serge WEMBE II-ESSOUMBA · Cheik LAWANI · Javier LADINO*
