# Système de Veille Stratégique : Cloud, On-Premise & Hybride (2026)

## Problématique
Comment l'équation entre la maîtrise des coûts (**FinOps**), l'urgence écologique (**Green IT**) et la **Souveraineté des données** redessine-t-elle les frontières entre le Cloud, le On-Premise et l'Hybride pour les années à venir ?.

## Équipe
* **Serge WEMBE II**
* **Cheik ANAIS** 
* **Javier LADINO**

## Stack Technique
* **Collecte :** Feedly, Google Alerts, Talkwalker.
* **Automatisation :** Make.com / Zapier.
* **Stockage & Analyse :** Notion, Google Drive.
* **Communication :** Slack, Microsoft Teams.

## Structure du Dépôt
Consultez le dossier `docs/` pour la méthodologie et `config/` pour les paramètres d'automatisation.

[Fiche de suivi Dossier Veille](https://docs.google.com/document/d/15UXsffiuzuDaxjrri_YVXkRCRfsJbNPs/edit?usp=sharing&ouid=115089839475756212140&rtpof=true&sd=true)
___

## Projet : Système de Veille Stratégique IT (FinOps, Green IT & Souveraineté)

### **1. Charte du Projet et Problématique**
Ce système de veille vise à analyser comment l'équilibre entre la maîtrise des coûts (FinOps), l'urgence écologique (Green IT) et la Souveraineté des données redéfinit les infrastructures Cloud et On-Premise pour 2026.

**Objectifs clés :**
• Anticiper les mouvements de Cloud Repatriation.

• Évaluer le TCO (Total Cost of Ownership) réel face aux coûts cachés du Cloud.

• Suivre les évolutions réglementaires (RGPD, Cloud Act) impactant la souveraineté.

• Intégrer les métriques d'éco-responsabilité dans le choix des infrastructures.

### **2. Architecture Technique de l'Automation (Make / Zapier)**
Pour éviter la surcharge informationnelle, nous articulons le flux de données via Make.com (recommandé pour sa granularité).

**Configuration du Scénario "Automated Intelligence Pipeline" :**

**1. Trigger (Source) :** Watch RSS Feeds via Feedly ou requêtes Google Alerts agrégées.

**2. Filter (Logique métier) :** * Le système ne laisse passer que les articles contenant des mots-clés spécifiques : FinOps, Green IT, Souveraineté, Hybrid Cloud, Repatriation.

    ◦ Exclusion automatique des contenus sponsorisés ou sans date.

**3. Router (Analyse de pertinence) :**

    ◦ Branche A (Urgent) : Si mot-clé "Faille de sécurité" ou "Changement tarifaire majeur" → Notification immédiate sur Slack (Canal #veille-critique).

    ◦ Branche B (Standard) : Envoi vers une base de données Notion pour analyse approfondie.

**4. Action (Stockage) :** Création d'une page dans le "Wiki de Veille" Notion avec résumé automatique et tags sectoriels (Bancaire, Santé, Public).

**3. Structure du Répertoire (GitHub)**
Voici l'organisation recommandée pour votre dépôt Git afin d'assurer la pérennité et la collaboration du groupe.

## 3. Structure du Répertoire (GitHub)

Voici l'organisation recommandée pour votre dépôt Git afin d'assurer la pérennité et la collaboration du groupe.

```
├── .github/                # Workflows pour automatiser certaines tâches de doc
├── config/                 # Configurations techniques
│   ├── make_scenario.json  # Export du blueprint Make.com
│   ├── feedly_opml.xml     # Liste des flux RSS exportée
│   └── alerts_setup.md     # Liste des requêtes Google Alerts paramétrées
├── docs/                   # Méthodologie et cadre stratégique
│   ├── problematique.md   # Détails de la thématique
│   ├── sources.md         # Liste exhaustive des sites et experts suivis
│   └── kpi_metrics.md      # Indicateurs de performance de la veille
├── templates/              # Modèles de documents
│   ├── monthly_report.md   # Template de synthèse mensuelle
│   └── decision_matrix.md  # Matrice de décision Cloud vs On-Prem
├── reports/                # Archives des rapports produits (par mois)
│   └── 2026/
│       └── 03-mars-report.md
├── README.md              # Présentation générale du projet
└── .gitignore              # Exclusion des fichiers sensibles ou temporaires
```

## 4. Guide d'Installation et de Versioning (Git)

Pour initialiser ce système sur votre machine et le lier à GitHub :

### Initialisation du dépôt

````
# Créer le dossier du projet
mkdir veille_techno_2026
cd veille_techno_2026

# Initialiser Git
git init

# Créer la structure de base
mkdir config docs templates reports
touch README.md
````


### Routine de mise à jour (Workflow de groupe)

Pour chaque nouvelle analyse ou mise à jour de source:

1. **Pull :** Toujours récupérer les dernières recherches des autres membres (`git pull origin main`).
2. **Add :** Ajouter vos rapports ou nouveaux flux (`git add .`).
3. **Commit :** Documenter le changement (`git commit -m "Ajout analyse TCO Cloud 2026 - Secteur Bancaire"`).
4. **Push :** Partager sur GitHub (`git push origin main`).

---

## 5. Planning Opérationnel de Diffusion

Le système ne s'arrête pas à la collecte ; il suit un cycle de diffusion strict pour influencer les décisions:

- **Quotidien :** Veille flash sur Slack (Alertes critiques).
- **Hebdomadaire :** Sélection des 5 meilleurs articles dans Notion pour lecture différée.
- **Mensuel :** Rapport visuel (Infographie via Canva) synthétisant les tendances de coûts et d'écologie IT.
- **Trimestriel :** Présentation stratégique aux décideurs basée sur les données collectées.

---

## 6. Gestion des Risques de Veille

Pour garantir la qualité, le projet intègre des garde-fous contre les biais identifiés :

- **Biais des fournisseurs :** Une source "fournisseur" (AWS/Azure) doit toujours être croisée avec une analyse indépendante (Gartner/Forrester).
- **Obsolescence :** Tout article de plus de 6 mois est marqué comme "archive" pour éviter des décisions basées sur des prix périmés.


