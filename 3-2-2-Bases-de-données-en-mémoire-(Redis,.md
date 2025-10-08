# Bases de données en mémoire (Redis, Memcached) pour la rapidité en temps réel

## 1. Pourquoi utiliser une base en mémoire ?

Les bases de données en mémoire stockent les données directement dans la RAM, ce qui offre des temps d’accès très faibles (ordres de microsecondes à millisecondes). Cette caractéristique est essentielle pour les applications temps réel nécessitant un traitement ultra-rapide, comme la gestion de sessions, le caching ou les systèmes de messagerie instantanée.

---

## 2. Redis : base de données en mémoire polyvalente

### Fonctionnalités

- Stockage clé-valeur mais aussi structures complexes : listes, sets, sorted sets, hashes.
- Supporte la **persistance** sur disque via snapshots (RDB) ou enregistrement append-only (AOF).
- Fonctionnalités avancées de publication/abonnement (Pub/Sub).
- Transactions, scripts Lua, et haute disponibilité via Redis Sentinel ou cluster.
- Très utilisée pour le caching, la gestion de sessions, les files de messages temporaires.

### Exemples d’utilisation

```javascript
const Redis = require('ioredis');
const redis = new Redis();

// Incrémenter un compteur en temps réel
redis.incr('nombre_visites');

// Stocker un objet JSON sérialisé dans une hash
redis.hset('utilisateur:42', 'nom', 'Alice', 'age', '30');

// Publier un événement
redis.publish('chatroom', 'Nouveau message');
```

---

## 3. Memcached : cache mémoire simple et performant

### Fonctionnalités

- Stockage clé-valeur en mémoire, très rapide.
- Pas de persistance native : si serveur redémarre, données perdues.
- Orienté cache, souvent placé devant une base de données lente.
- Simplicité d’installation et d’utilisation.
- Supporte clustering via clients ou solutions tierces.

### Exemple simple en Node.js

```javascript
const memjs = require('memjs');
const mc = memjs.Client.create();

mc.set('clé', 'valeur', { expires: 300 }, (err, success) => {
  if(success) console.log('Valeur mise en cache');
});

mc.get('clé', (err, val) => {
  if(val) console.log('Valeur cache:', val.toString());
});
```

---

## 4. Comparaison Redis vs Memcached

| Critère               | Redis                                  | Memcached                    |
|-----------------------|--------------------------------------|------------------------------|
| Structures de données  | Clé-valeur + Structures complexes    | Clé-valeur simple            |
| Persistance           | Oui (RDB, AOF)                        | Non                         |
| Fonctionnalités       | Pub/Sub, scripts Lua, transactions    | Simple cache                 |
| Scalabilité           | Cluster natif                        | Clustering tiers             |
| Cas d’usage           | Cache, sessions, files temporaires, message queue | Simple cache, accélération accès BD |

---

## 5. Cas d’usage typiques en temps réel

- **Redis** : sessions utilisateurs, leaderboard, diffusion d’événements temps réel, files de messages.
- **Memcached** : cache des résultats de requêtes coûteuses, réduction de latence des bases relationnelles.

---

## 6. Architecture simplifiée avec Redis et Memcached

```mermaid
graph TD
    Utilisateur -->|Requête rapide| API[Serveur API]

    API --> Redis[Redis - Cache & Pub/Sub]
    API --> Memcached[Memcached - Cache simple]

    API --> Backend[Base de données principale (MongoDB, PostgreSQL)]
```

---

## 7. Sources

- Redis Documentation – [https://redis.io/docs/](https://redis.io/docs/)  
- Memcached Official Site – [https://memcached.org/](https://memcached.org/)  
- ioredis GitHub – [https://github.com/luin/ioredis](https://github.com/luin/ioredis)  
- Node Memjs – [https://github.com/memcachier/memjs](https://github.com/memcachier/memjs)  
- DigitalOcean – [How to Use Redis and Memcached](https://www.digitalocean.com/community/tutorials/how-to-use-redis-and-memcached-fundamentals)

---

Les bases en mémoire apportent la réactivité nécessaire pour les systèmes temps réel en fournissant des accès fulgurants aux données fréquemment consultées ou en transit. Redis, par sa richesse fonctionnelle, offre une palette étendue d’outils, tandis que Memcached reste une solution légère et optimisée pour le cache simple. Le choix dépendra des besoins spécifiques en vitesse, structures et persistance.