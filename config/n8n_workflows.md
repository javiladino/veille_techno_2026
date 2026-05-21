# Documentation des workflows n8n — Veille EPSI 2026

> **Outil** : n8n (self-hosted, npm global)
> **Version** : n8n latest
> **Déploiement** : VM Proxmox · Debian 12 · Service systemd
> **Accès** : SSH tunnel → `http://localhost:5678`

---

## Configuration du service systemd

Fichier `/etc/systemd/system/n8n.service` :

```ini
[Unit]
Description=n8n Workflow Automation
After=network.target

[Service]
Type=simple
User=jladino
Environment=N8N_PORT=5678
Environment=N8N_HOST=127.0.0.1
Environment=N8N_PROTOCOL=http
Environment=WEBHOOK_URL=http://localhost:5678/
ExecStart=/usr/bin/n8n start
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
# Commandes de gestion
sudo systemctl enable n8n
sudo systemctl start n8n
sudo systemctl status n8n
```

---

## Accès depuis le poste client

```bash
# Tunnel SSH (maintenir la session ouverte)
ssh -L 5678:127.0.0.1:5678 jladino@172.16.89.44

# Accès dans le navigateur
http://localhost:5678/
```

---

## Workflow 1 — Collecte des articles

**Nom** : `Collecte Veille`
**Déclenchement** : toutes les 2 heures
**Fonction** : récupère les articles non lus dans Miniflux, filtre par thématique, sauvegarde en Google Drive, notifie Slack

### Nœuds

#### Nœud 1 — Schedule Trigger

```
Trigger Interval : Hours
Hours Between Triggers : 2
```

#### Nœud 2 — HTTP Request (Miniflux)

```
Method  : GET
URL     : http://127.0.0.1:8080/v1/entries?status=unread&limit=50
Headers :
  X-Auth-Token : <MINIFLUX_API_KEY>
```

#### Nœud 3 — Split Out

```
Field To Split Out : entries
```

#### Nœud 4 — Code (filtrage par mots-clés)

```javascript
const entry = $input.item.json;
const title = (entry.title || '').toLowerCase();
const content = (entry.summary || entry.content || '').toLowerCase();
const text = title + ' ' + content;

const keywords = [
  // FinOps
  'finops', 'cloud cost', 'tco', 'cost optimization', 'cloud spending',
  'capex', 'opex', 'cloud billing', 'savings plan', 'coût du cloud',
  // Green IT
  'green it', 'énergie', 'datacenter', 'empreinte carbone', 'sustainability',
  'carbon', 'renewable energy', 'sobriété numérique', 'consommation énergétique',
  // Souveraineté
  'sovereignty', 'souveraineté', 'gdpr', 'rgpd', 'cloud repatriation',
  'ai act', 'données personnelles', 'conformité', 'compliance', 'vie privée'
];

const matched = keywords.some(kw => text.includes(kw));
if (!matched) return [];

return { json: entry };
```

#### Nœud 5 — Code (construction markdown)

```javascript
const entry = $input.item.json;

const date = new Date(entry.published_at).toISOString().split('T')[0];
const title = (entry.title || 'Sans titre').replace(/"/g, "'");
const url = entry.url || '';
const source = entry.feed?.title || 'Inconnu';
const summary = entry.summary?.replace(/<[^>]*>/g, '').trim() ||
                entry.content?.replace(/<[^>]*>/g, '').substring(0, 600).trim() ||
                'Pas de résumé disponible.';
const fileName = `article_${date}_${entry.id}.md`;

const markdown = `---
date: ${date}
source: ${source}
titre: "${title}"
url: ${url}
tags: []
---

## Résumé

${summary}

## Source originale

${url}
`;

const binaryData = await this.helpers.prepareBinaryData(
  Buffer.from(markdown, 'utf-8'),
  fileName,
  'text/plain'
);

return {
  json: { file_name: fileName, entry_id: entry.id },
  binary: { data: binaryData }
};
```

#### Nœud 6 — Google Drive (upload)

```
Credential  : Google Drive account
Resource    : File
Operation   : Upload
Field Name  : data
File Name   : {{ $json.file_name }}
Parent Drive: My Drive
Parent Folder: /01_Raw/
```

#### Nœud 7 — HTTP Request (marquer lu dans Miniflux)

```
Method      : PUT
URL         : http://127.0.0.1:8080/v1/entries
Headers     :
  X-Auth-Token : <MINIFLUX_API_KEY>
  Content-Type : application/json
Body (JSON) :
{
  "entry_ids": [{{ $json.entry_id }}],
  "status": "read"
}
```

#### Nœud 8 — HTTP Request (Slack #veille-critique)

```
Method      : POST
URL         : <SLACK_WEBHOOK_VEILLE_CRITIQUE>
Body (JSON) :
{
  "text": "🚨 *Nouvel article veille*\n*{{ $json.file_name }}*\nArchivé dans Google Drive /01_Raw/"
}
```

---

## Workflow 2 — Digest hebdomadaire

**Nom** : `Digest Hebdomadaire`
**Déclenchement** : chaque lundi à 08h00
**Fonction** : agrège les articles de la semaine par thématique, génère un digest `.md`, notifie Slack

### Nœuds

#### Nœud 1 — Schedule Trigger

```
Trigger Interval : Weeks
Day of Week      : Monday
Hour             : 8
Minute           : 0
```

#### Nœud 2 — HTTP Request (Miniflux — 7 derniers jours)

```
Method  : GET
URL     : http://127.0.0.1:8080/v1/entries?limit=100&after={{ Math.floor((Date.now() - 7*24*60*60*1000) / 1000) }}
Headers :
  X-Auth-Token : <MINIFLUX_API_KEY>
```

#### Nœud 3 — Code (construction du digest)

```javascript
const data = $input.first().json;
const entries = data.entries || [];

const now = new Date();
const date = now.toISOString().split('T')[0];
const weekNumber = Math.ceil(
  (now - new Date(now.getFullYear(), 0, 1)) / (7 * 24 * 60 * 60 * 1000)
);

const themes = { finops: [], greenit: [], sovereignty: [], other: [] };

const finopsKw = ['finops', 'cloud cost', 'tco', 'cost optimization', 'capex',
  'opex', 'cloud billing', 'coût du cloud', 'budget cloud', 'savings plan'];
const greenKw = ['green it', 'greenit', 'énergie', 'datacenter', 'data center',
  'empreinte carbone', 'carbon footprint', 'sustainability', 'carbon', 'renewable',
  'climate', 'consommation énergétique', 'transition écologique', 'sobriété numérique',
  'environmental', 'ecological'];
const sovKw = ['sovereignty', 'souveraineté', 'gdpr', 'rgpd', 'cloud repatriation',
  'rapatriement', 'ai act', 'data protection', 'protection des données', 'cloud act',
  'vie privée', 'données personnelles', 'réglementation', 'conformité', 'compliance',
  'sovereign', 'censure', 'surveillance'];

for (const entry of entries) {
  const text = [
    entry.title || '',
    (entry.summary || '').replace(/<[^>]*>/g, ''),
    (entry.content || '').replace(/<[^>]*>/g, '').substring(0, 800)
  ].join(' ').toLowerCase();

  if (finopsKw.some(k => text.includes(k))) themes.finops.push(entry);
  else if (greenKw.some(k => text.includes(k))) themes.greenit.push(entry);
  else if (sovKw.some(k => text.includes(k))) themes.sovereignty.push(entry);
  else themes.other.push(entry);
}

const formatArticles = (list) => list.slice(0, 5).map(e =>
  `### ${e.title}\n**Source** : ${e.feed?.title || 'Inconnu'} | **Date** : ${e.published_at?.split('T')[0]}\n**URL** : ${e.url}\n\n${(e.summary || '').replace(/<[^>]*>/g, '').substring(0, 300)}\n`
).join('\n---\n\n');

const digest = `# Digest Veille Technologique — Semaine ${weekNumber}
**Période** : semaine du ${date}
**Total articles** : ${entries.length}

---

## 💰 FinOps (${themes.finops.length} articles)

${themes.finops.length > 0 ? formatArticles(themes.finops) : '_Aucun article cette semaine._'}

---

## 🌿 Green IT (${themes.greenit.length} articles)

${themes.greenit.length > 0 ? formatArticles(themes.greenit) : '_Aucun article cette semaine._'}

---

## 🔐 Souveraineté des données (${themes.sovereignty.length} articles)

${themes.sovereignty.length > 0 ? formatArticles(themes.sovereignty) : '_Aucun article cette semaine._'}

---

## 📌 Autres (${themes.other.length} articles)

${themes.other.length > 0 ? formatArticles(themes.other) : '_Aucun article._'}

---

*Digest généré automatiquement — Veille EPSI 2026*
`;

const fileName = `digest_semaine_${weekNumber}_${date}.md`;

const binaryData = await this.helpers.prepareBinaryData(
  Buffer.from(digest, 'utf-8'),
  fileName,
  'text/plain'
);

return {
  json: {
    file_name: fileName,
    week_number: weekNumber,
    total_articles: entries.length,
    finops_count: themes.finops.length,
    greenit_count: themes.greenit.length,
    sovereignty_count: themes.sovereignty.length,
    other_count: themes.other.length
  },
  binary: { data: binaryData }
};
```

#### Nœud 4 — Google Drive (upload)

```
Credential  : Google Drive account
Resource    : File
Operation   : Upload
Field Name  : data
File Name   : {{ $json.file_name }}
Parent Drive: My Drive
Parent Folder: /02_Digest/
```

#### Nœud 5 — HTTP Request (Slack #veille-hebdo)

```
Method      : POST
URL         : <SLACK_WEBHOOK_VEILLE_HEBDO>
Body (JSON) :
{
  "text": "📊 *Digest Veille — Semaine {{ $json.week_number }}*\n\n💰 FinOps : {{ $json.finops_count }} articles\n🌿 Green IT : {{ $json.greenit_count }} articles\n🔐 Souveraineté : {{ $json.sovereignty_count }} articles\n\n📁 Fichier : {{ $json.file_name }}\n\n*Action requise :*\n1. Ouvrir NotebookLM\n2. Ajouter le fichier depuis Google Drive /02_Digest/\n3. Générer l'Audio Overview\n4. Déposer le .m4a dans Google Drive /03_Podcast/"
}
```

---

## Workflow 3 — Notification podcast

**Nom** : `Notification Podcast`
**Déclenchement** : création d'un fichier dans `/03_Podcast/`
**Fonction** : détecte automatiquement un nouveau fichier audio et notifie Slack
**Statut** : doit rester **actif en permanence**

### Nœuds

#### Nœud 1 — Google Drive Trigger

```
Credential      : Google Drive account
Event           : Changes involving a Specific Folder
Trigger On      : File Created
Drive           : My Drive
Folder          : /03_Podcast/
Recursive       : false
```

#### Nœud 2 — HTTP Request (Slack #veille-podcast)

```
Method      : POST
URL         : <SLACK_WEBHOOK_VEILLE_PODCAST>
Body (JSON) :
{
  "text": "🎙️ *Nouveau Podcast Veille disponible !*\n\n📁 Fichier : {{ $json.name }}\n📅 Date : {{ $json.createdTime }}\n\n🔗 Écouter : {{ $json.webViewLink }}\n\n_Généré depuis le Digest hebdomadaire — Veille EPSI 2026_ 🎧"
}
```

---

## Dépannage

### n8n ne démarre pas

```bash
sudo journalctl -u n8n -n 50 --no-pager
sudo systemctl restart n8n
```

### Erreur OAuth Google Drive

Le tunnel SSH doit être actif avec `WEBHOOK_URL=http://localhost:5678/`.
Vérifier la configuration du service :

```bash
sudo systemctl cat n8n | grep WEBHOOK_URL
# Doit retourner : WEBHOOK_URL=http://localhost:5678/
```

### Workflow 1 — aucun article filtré

Vérifier que Miniflux a des articles non lus :

```bash
curl -s http://127.0.0.1:8080/v1/entries?status=unread \
  -H "X-Auth-Token: <API_KEY>" | python3 -m json.tool | grep total
```

### Workflow 2 — catégories vides

Activer "Fetch original content" sur chaque flux dans Miniflux pour enrichir le contenu indexé.

---

*Documentation n8n — Projet veille technologique EPSI 2025-2026*
