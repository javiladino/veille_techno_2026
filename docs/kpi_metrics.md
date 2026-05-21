# Indicateurs de performance — Veille EPSI 2026

> **Version** : POC v2 — opérationnel depuis mai 2026
> **Mesure** : hebdomadaire, via Google Drive et Slack analytics

---

## KPIs opérationnels

| Indicateur | Cible | Outil de mesure | Statut |
|---|---|---|---|
| Sources RSS actives | ≥ 10 flux | Miniflux → Feeds | ✅ 10+ actifs |
| Articles collectés / semaine | 50–100 | n8n logs / Digest | ✅ ~98 semaine 21 |
| Articles catégorisés / semaine | ≥ 15 | Section digest | ✅ 14 semaine 21 |
| Articles FinOps / semaine | ≥ 3 | Digest /02_Digest/ | 🔧 1 semaine 21 |
| Articles Green IT / semaine | ≥ 3 | Digest /02_Digest/ | ✅ 4 semaine 21 |
| Articles Souveraineté / semaine | ≥ 3 | Digest /02_Digest/ | ✅ 9 semaine 21 |
| Digest généré / semaine | 1 | Google Drive /02_Digest/ | ✅ Automatique |
| Podcast produit / semaine | 1 | Google Drive /03_Podcast/ | ✅ Manuel ~4 min |
| Alertes Slack (#veille-critique) | Temps réel | Slack | ✅ Opérationnel |
| Disponibilité Miniflux | > 99% | `systemctl status` | ✅ Actif |
| Disponibilité n8n | > 99% | `systemctl status` | ✅ Actif |
| Coût infrastructure mensuel | 0€ | — | ✅ 0€ |

---

## KPIs qualitatifs

| Indicateur | Description | Fréquence |
|---|---|---|
| Pertinence des articles | % articles correctement catégorisés | Hebdomadaire |
| Qualité du digest | Richesse des résumés, complétude | Hebdomadaire |
| Qualité du podcast | Clarté, couverture thématique | Hebdomadaire |
| Couverture thématique | Équilibre FinOps / Green IT / Souveraineté | Mensuel |

---

## Résultats observés — Semaine 21 (19–21 mai 2026)

```
Total articles collectés  : 98
├── 💰 FinOps             :  1  (1%)
├── 🌿 Green IT           :  4  (4%)
├── 🔐 Souveraineté       :  9  (9%)
└── 📌 Autres             : 84 (86%)

Digest généré             : digest_semaine_21_2026-05-21.md ✅
Podcast généré            : L_IA_du_G20_aux_serveurs_marins.m4a ✅
Notification Slack        : #veille-hebdo + #veille-podcast ✅
```

**Axe d'amélioration identifié** : renforcer les sources FinOps spécialisées
pour améliorer la couverture (cible ≥ 5 articles/semaine).

---

## Évolution cible — Phase 4

| Indicateur | Actuel | Cible phase 4 |
|---|---|---|
| Sources RSS actives | 10 | 20+ |
| Articles FinOps / semaine | 1 | 5+ |
| Articles avec résumé complet | ~20% | 80% |
| Catégories Autres / total | 86% | < 50% |
| Rapports mensuels | Manuel | Automatisé |

---

*KPIs — Projet veille technologique EPSI 2025-2026*
