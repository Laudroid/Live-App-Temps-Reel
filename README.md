# Cours App Temps Reel

## Objectifs de la formation

- Concevoir une application Web temps réel de bout en bout
- Choisir la technologie adaptée (WebSocket, Server-Sent Events, polling long, etc.)
- Déployer une architecture adaptée à la charge et à la latence
- Utiliser des outils comme Socket.IO, Redis, Kafka, Firebase, ou Supabase
- Implémenter un système de messagerie, de notification ou de collaboration en temps réel
## Séance 1 : Introduction aux applications temps réel et technologies fondamentales

### Objectifs pédagogiques

- Comprendre les concepts fondamentaux des applications Web temps réel.
- Différencier et choisir entre le polling, les Server-Sent Events (SSE) et les WebSockets.
- Mettre en œuvre des exemples basiques de chaque technologie.
### Contenus

#### Théorie : Qu'est-ce qu'une application temps réel ?

- Définitions, cas d'usage (chat, jeux, collaboration, trading).
- Les défis du temps réel (latence, persistance, scalabilité).
- Évolution des communications web (HTTP traditionnel vs. connexions persistantes).
#### Théorie : Les paradigmes de communication temps réel

- Polling et Long Polling : principes, avantages et inconvénients.
- Server-Sent Events (SSE) : communication unidirectionnelle du serveur au client, cas d'usage.
- WebSockets : communication bidirectionnelle full-duplex, handshake, avantages clés.
## Séance 2 : Maîtrise des WebSockets et Socket.IO

### Objectifs pédagogiques

- Approfondir l'utilisation des WebSockets natifs.
- Comprendre et utiliser Socket.IO pour des applications temps réel robustes.
- Implémenter une application de chat multi-utilisateurs avec Socket.IO.
### Contenus

#### Théorie : API WebSockets et gestion des connexions

- Utilisation des API WebSocket côté client (JavaScript) et côté serveur (ex: `ws` en Node.js).
- Gestion des événements de connexion, déconnexion, message.
- Problématiques des WebSockets natifs (reconnexion, fallbacks, diffusion).
#### Théorie : Introduction à Socket.IO

- Pourquoi Socket.IO ? (fiabilité, reconnexion automatique, transport fallbacks).
- Concepts clés : Événements, namespaces, rooms.
- Diffusion de messages (broadcast, to, emit).
#### TP : Construction d'une application de chat temps réel avec Socket.IO

- Initialisation du serveur et du client Socket.IO.
- Gestion des utilisateurs et de leur présence.
- Envoi et réception de messages dans des salons (rooms).
- Implémentation de messages privés (optionnel).
## Séance 3 : Architectures et persistance pour le temps réel

### Objectifs pédagogiques

- Comprendre les défis de scalabilité des applications temps réel.
- Intégrer des systèmes de messagerie (Redis Pub/Sub, Kafka) pour la distribution des événements.
- Appliquer des patterns architecturaux pour la haute disponibilité.
### Contenus

#### Théorie : Architectures distribuées pour le temps réel

- Défis de la scalabilité horizontale (sticky sessions vs. state sharing).
- Introduction aux Brokers de messages (message queues, publish/subscribe).
- Concepts de Redis Pub/Sub pour la diffusion d'événements.
- Introduction à Apache Kafka (topics, producers, consumers) pour les flux de données à grande échelle.
#### Théorie : Persistance des données temps réel

- Bases de données NoSQL (MongoDB, Cassandra) pour la flexibilité.
- Bases de données en mémoire (Redis, Memcached) pour la vitesse.
- Stratégies de stockage des événements (event sourcing).
#### TP : Scalabilité avec Redis Pub/Sub et Socket.IO

- Mise en place de plusieurs instances de serveur Socket.IO derrière un load balancer.
- Configuration de Redis en tant qu'adaptateur Socket.IO pour synchroniser les événements entre les instances.
- Tester la diffusion de messages entre clients connectés à différentes instances de serveur.
## Séance 4 : Services Cloud et collaboration en temps réel

### Objectifs pédagogiques

- Découvrir les services cloud pour le développement d'applications temps réel.
- Utiliser Firebase ou Supabase pour construire des fonctionnalités temps réel.
- Implémenter un système de collaboration temps réel ou de notifications.
- Aborder les aspects de sécurité spécifiques au temps réel.
### Contenus

#### Théorie : Services temps réel managés

- Firebase (Realtime Database, Firestore) : présentation, modèles de données, sécurité.
- Supabase Realtime : PostgreSQL en temps réel, écoute des changements de base de données.
- Comparaison et cas d'usage des différentes plateformes.
- Webhooks vs. WebSockets : quand utiliser quoi ?
#### Théorie : Patterns avancés et sécurité

- Introduction aux CRDTs (Conflict-free Replicated Data Types) pour la collaboration.
- Sécurité des applications temps réel : authentification, autorisation, prévention des attaques DDoS, chiffrement.
- Gestion des identités et des sessions dans un environnement temps réel.
#### TP : Application collaborative avec Firebase/Supabase

- Création d'une application de tableau blanc collaboratif simple ou d'un éditeur de texte en temps réel.
- Utilisation des fonctionnalités de synchronisation automatique de Firebase Realtime Database/Firestore ou Supabase Realtime.
- Gestion des droits d'accès (règles de sécurité Firebase/Row Level Security Supabase).
## Séance 5 : Déploiement, monitoring et bonnes pratiques

### Objectifs pédagogiques

- Connaître les stratégies de déploiement des applications temps réel.
- Apprendre à surveiller les performances et la santé des applications temps réel.
- Comprendre les bonnes pratiques pour le développement et la maintenance.
- Révision et synthèse des différentes technologies et choix architecturaux.
### Contenus

#### Théorie : Déploiement et Opérations (DevOps)

- Options de déploiement (VPS, PaaS comme Heroku/Vercel, conteneurisation avec Docker/Kubernetes).
- Gestion des connexions persistantes sur des infrastructures cloud.
- Monitoring : métriques clés (connexions, latence, erreurs), outils (Prometheus, Grafana, ELK Stack).
- Gestion des logs et traçabilité des événements en temps réel.
#### Théorie : Bonnes pratiques et cas d'usage avancés

- Gestion des erreurs et des reconnexions côté client et serveur.
- Optimisation des performances (compression, batching, throttling).
- Recommandations pour le choix des technologies en fonction des besoins.
- Cas d'usage avancés (IoT, streaming vidéo, réalité augmentée en temps réel).
#### TP : Déploiement et monitoring d'une application temps réel

- Déploiement de l'application de chat ou collaborative sur une plateforme cloud (ex: Heroku, un VPS configuré).
- Configuration d'un outil de monitoring basique (ex: New Relic, ou logs console avancés).
- Test de la résilience de l'application face à des déconnexions/reconnexions simulées.
