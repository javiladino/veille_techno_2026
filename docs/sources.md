# Cartographie des sources de veille — EPSI 2026

> **Mise à jour** : mai 2026
> **Statut** : sources actives dans Miniflux (VM Proxmox)

---

## Sources actives par thématique

### 💰 FinOps

| Source | Type | URL RSS | Statut |
|---|---|---|---|
| FinOps Foundation | Institutionnel | `https://www.finops.org/feed/` | ✅ Actif |
| Le Monde Informatique — Infrastructure | Presse IT FR | `https://www.lemondeinformatique.fr/flux-rss/thematique/infrastructure/rss.xml` | ✅ Actif |
| Silicon.fr | Presse IT FR | `https://www.silicon.fr/feed` | ✅ Actif |
| The New Stack | Presse IT EN | `https://thenewstack.io/feed/` | ✅ Actif |

**Mots-clés de détection :**
`finops`, `cloud cost`, `tco`, `cost optimization`, `capex`, `opex`,
`cloud billing`, `coût du cloud`, `budget cloud`, `savings plan`

---

### 🌿 Green IT

| Source | Type | URL RSS | Statut |
|---|---|---|---|
| GreenIT.fr | Spécialisé FR | `https://www.greenit.fr/feed/` | ✅ Actif |
| The Shift Project | Think tank FR | `https://theshiftproject.org/feed/` | ✅ Actif |
| ADEME | Institutionnel FR | `https://www.ademe.fr/rss.xml` | ✅ Actif |
| Next.ink | Presse IT FR | `https://next.ink/feed/` | ✅ Actif |
| IEA Energy | Institutionnel EN | `https://www.iea.org/rss/news.xml` | ❌ URL invalide |

**Mots-clés de détection :**
`green it`, `énergie`, `datacenter`, `empreinte carbone`, `carbon footprint`,
`sustainability`, `carbon`, `renewable energy`, `sobriété numérique`,
`consommation énergétique`, `transition écologique`, `environmental`

---

### 🔐 Souveraineté des données

| Source | Type | URL RSS | Statut |
|---|---|---|---|
| CNIL | Autorité publique FR | `https://www.cnil.fr/fr/rss.xml` | ✅ Actif |
| Next.ink | Presse IT FR | `https://next.ink/feed/` | ✅ Actif |
| Silicon.fr | Presse IT FR | `https://www.silicon.fr/feed` | ✅ Actif |
| Numerama | Presse tech FR | `https://www.numerama.com/feed/` | ✅ Actif |

**Mots-clés de détection :**
`sovereignty`, `souveraineté`, `gdpr`, `rgpd`, `cloud repatriation`,
`rapatriement`, `ai act`, `data protection`, `protection des données`,
`cloud act`, `vie privée`, `conformité`, `compliance`, `réglementation`

---

## Sources à intégrer (phase 4)

### FinOps

| Source | URL RSS |
|---|---|
| CloudOptimizer | `https://cloudoptimizer.net/feed/` |
| InfoQ Cloud | `https://feed.infoq.com/cloud` |

### Green IT

| Source | URL RSS |
|---|---|
| WWF France | `https://www.wwf.fr/feed` |
| INRIA Actualités | `https://www.inria.fr/fr/rss.xml` |

### Souveraineté

| Source | URL RSS |
|---|---|
| European Commission Digital | `https://digital-strategy.ec.europa.eu/en/rss.xml` |
| ENISA | `https://www.enisa.europa.eu/rss-feeds/rss.xml` |

---

## Sources académiques et institutionnelles

| Source | Domaine | URL |
|---|---|---|
| Gartner | Analyse marché | Publication manuelle (pas de RSS libre) |
| Forrester | Analyse marché | Publication manuelle |
| IDC | Études sectorielles | Publication manuelle |
| European Commission Digital | Réglementation EU | `https://digital-strategy.ec.europa.eu/en/rss.xml` |

---

## Import OPML

Fichier d'import pour Miniflux (`Feeds → Import`) :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<opml version="2.0">
  <head>
    <title>Veille EPSI 2026 — Flux RSS</title>
    <dateCreated>2026-05-21</dateCreated>
  </head>
  <body>
    <outline text="FinOps" title="FinOps">
      <outline type="rss" text="FinOps Foundation" title="FinOps Foundation"
        xmlUrl="https://www.finops.org/feed/" htmlUrl="https://www.finops.org"/>
      <outline type="rss" text="Silicon.fr" title="Silicon.fr"
        xmlUrl="https://www.silicon.fr/feed" htmlUrl="https://www.silicon.fr"/>
      <outline type="rss" text="Le Monde Informatique" title="Le Monde Informatique"
        xmlUrl="https://www.lemondeinformatique.fr/flux-rss/thematique/infrastructure/rss.xml"
        htmlUrl="https://www.lemondeinformatique.fr"/>
    </outline>
    <outline text="Green IT" title="Green IT">
      <outline type="rss" text="GreenIT.fr" title="GreenIT.fr"
        xmlUrl="https://www.greenit.fr/feed/" htmlUrl="https://www.greenit.fr"/>
      <outline type="rss" text="The Shift Project" title="The Shift Project"
        xmlUrl="https://theshiftproject.org/feed/" htmlUrl="https://theshiftproject.org"/>
      <outline type="rss" text="ADEME" title="ADEME"
        xmlUrl="https://www.ademe.fr/rss.xml" htmlUrl="https://www.ademe.fr"/>
      <outline type="rss" text="Next.ink" title="Next.ink"
        xmlUrl="https://next.ink/feed/" htmlUrl="https://next.ink"/>
    </outline>
    <outline text="Souveraineté" title="Souveraineté des données">
      <outline type="rss" text="CNIL" title="CNIL"
        xmlUrl="https://www.cnil.fr/fr/rss.xml" htmlUrl="https://www.cnil.fr"/>
      <outline type="rss" text="Numerama" title="Numerama"
        xmlUrl="https://www.numerama.com/feed/" htmlUrl="https://www.numerama.com"/>
    </outline>
  </body>
</opml>
```

---

*Cartographie des sources — Projet veille technologique EPSI 2025-2026*
