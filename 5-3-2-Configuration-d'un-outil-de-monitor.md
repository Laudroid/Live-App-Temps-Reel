# Configuration d’un outil de monitoring basique pour une application temps réel

## 1. Introduction

Le monitoring d’une application temps réel est indispensable pour assurer sa disponibilité, suivre ses performances et détecter rapidement les anomalies. Parmi les solutions accessibles, New Relic offre une interface complète tandis que les logs console avancés permettent un suivi simple et granulé.

Cet article présente une configuration basique du monitoring avec New Relic ainsi que quelques bonnes pratiques autour des logs.

---

## 2. Monitoring avec New Relic

### 2.1 Présentation

New Relic est un outil de monitoring SaaS qui collecte des métriques sur les performances applicatives, l’infrastructure, les erreurs, et génère des tableaux de bord et alertes.

### 2.2 Mise en place avec une application Node.js

#### Étape 1 : inscription et création d’une clé API

- Créer un compte New Relic (https://newrelic.com/).
- Récupérer la clé de licence dans le tableau de bord.

#### Étape 2 : installation du package Node.js

```bash
npm install newrelic
```

#### Étape 3 : configuration du module

- Placer un fichier `newrelic.js` à la racine du projet (fournit par New Relic ou généré).

- Modifier la variable `license_key` avec la clé obtenue.

```javascript
exports.config = {
  app_name: ['NomAppChat'],
  license_key: 'VOTRE_LICENSE_KEY',
  logging: { level: 'info' },
};
```

- Importer New Relic **en tout premier** dans le fichier d’entrée `server.js` :

```javascript
require('newrelic');
const express = require('express');
const app = express();
// suite du code
```

### 2.3 Visualisation et alertes

- Les données remontent automatiquement vers le dashboard.
- Configuration d’alertes e-mail/SMS possible sur erreurs ou dégradation.

---

## 3. Logs console avancés

### 3.1 Importance des logs structurés

Les logs formattés permettent une analyse automatisée plus efficace. Utiliser des formats JSON facilite leur ingestion dans des systèmes de gestion de logs (ex : ELK Stack).

### 3.2 Exemple avec Winston

Installation :

```bash
npm install winston
```

Configuration de base :

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.Console(),
    // fichier optionnel
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});

module.exports = logger;
```

Utilisation dans l’application :

```javascript
const logger = require('./logger');

app.use((req, res, next) => {
  logger.info({ message: 'Requête reçue', url: req.url, method: req.method });
  next();
});
```

---

## 4. Diagramme Mermaid : flux du monitoring simplifié

```mermaid
flowchart TD
  AppServer[Application Node.js] --> Logging[Logs Formatés (Winston)]
  AppServer --> NewRelic[Agent New Relic]
  Logging --> LogAggregator[ELK / Graylog]
  NewRelic --> Dashboard[Dashboard New Relic]
  LogAggregator --> DashboardLogs[Dashboard Logs]
```

---

## 5. Bonnes pratiques

- Placer l’initialisation de New Relic avant tout dans l’application.
- Utiliser des niveaux de logs (info, warn, error) distincts.
- Structurer les logs pour faciliter l’analyse automatique.
- Ne pas logger d’informations sensibles.
- Configurer des alertes sur les erreurs critiques pour réaction rapide.

---

## 6. Sources et références

- Documentation New Relic Node.js : https://docs.newrelic.com/docs/agents/nodejs-agent/installation-configuration/install-nodejs-agent/  
- Winston Logger GitHub : https://github.com/winstonjs/winston  
- Guide structuration des logs JSON : https://www.elastic.co/blog/a-pragmatic-guide-to-logging-in-elasticsearch  
- ELK Stack overview : https://www.elastic.co/what-is/elk-stack  

---

Intégrer un outil de monitoring comme New Relic couplé à une gestion avancée des logs console offre une visibilité essentielle sur la santé d’une application temps réel et sert de base pour anticiper incidents et optimiser la performance.