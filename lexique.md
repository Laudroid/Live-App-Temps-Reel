**Application Web temps réel** : Application web capable de réagir et de traiter des informations de manière quasi instantanée, offrant une interactivité immédiate.

**WebSocket** : Protocole de communication web bidirectionnel (full-duplex) qui établit une connexion persistante entre client et serveur pour l'échange de données en temps réel.

**Server-Sent Events (SSE)** : Technologie de communication unidirectionnelle où le serveur envoie un flux d'événements à un client via une connexion HTTP, utilisée pour les mises à jour du serveur vers le client.

**Polling long** : Technique où le client envoie une requête au serveur qui la garde ouverte jusqu'à ce qu'il y ait des données à envoyer ou un délai d'attente expiré.

**Socket.IO** : Bibliothèque JavaScript qui facilite la construction d'applications temps réel robustes, offrant des fonctionnalités comme la reconnexion automatique, les fallbacks de transport et la gestion des rooms/namespaces, bâtie sur les WebSockets.

**Redis** : Base de données en mémoire, souvent utilisée pour sa vitesse, sa persistance des données et ses capacités de diffusion d'événements (Pub/Sub) entre instances d'applications.

**Kafka** : Plateforme distribuée de streaming d'événements, conçue pour gérer des flux de données à grande échelle grâce à ses concepts de topics, producers et consumers.

**Firebase** : Suite de services cloud de Google, incluant des bases de données temps réel (Realtime Database, Firestore) pour la synchronisation automatique et instantanée des données.

**Supabase** : Alternative open source à Firebase, offrant un backend as a service avec une base de données PostgreSQL qui permet d'écouter les changements de données en temps réel via Supabase Realtime.

**Messagerie** : Fonctionnalité permettant l'échange de messages instantanés entre utilisateurs ou composants au sein d'une application temps réel.

**Notification** : Système d'alertes ou d'informations envoyées instantanément aux utilisateurs dès qu'un événement pertinent se produit.

**Collaboration en temps réel** : Fonctionnalité permettant à plusieurs utilisateurs de travailler simultanément sur un même contenu ou une même interface, avec des mises à jour visibles par tous instantanément.

**Latence** : Délai de temps entre l'envoi d'une donnée et sa réception, un défi majeur dans les applications temps réel.

**Persistance** : Capacité d'un système à stocker des données de manière durable pour qu'elles restent disponibles après la fin d'un processus ou une coupure.

**Scalabilité** : Capacité d'un système à gérer une charge de travail croissante ou à s'adapter à une augmentation de ses besoins en ressources, y compris horizontalement.

**Polling** : Technique de communication où le client interroge régulièrement le serveur pour vérifier la présence de nouvelles données.

**Full-duplex** : Mode de communication permettant la transmission simultanée de données dans les deux sens sur une même connexion.

**Handshake** : Processus initial d'établissement d'une connexion, où client et serveur s'accordent sur les paramètres de communication, notamment pour les WebSockets.

**Événements (Socket.IO)** : Messages ou actions déclenchées et transmises au sein de Socket.IO, permettant la communication entre le client et le serveur.

**Namespaces (Socket.IO)** : Mécanisme de Socket.IO permettant d'isoler des canaux de communication logiques au sein d'une même connexion WebSocket.

**Rooms (Socket.IO)** : Fonctionnalité de Socket.IO permettant de grouper des clients pour leur envoyer des messages collectivement, utile pour les chats ou les collaborations.

**Diffusion de messages** : Envoi d'un message à plusieurs clients simultanément, soit à tous les clients connectés, soit à ceux d'une room spécifique (aussi appelé Broadcast).

**API WebSockets** : Interface de programmation permettant d'interagir avec la technologie WebSocket côté client (JavaScript) et côté serveur (ex: `ws` en Node.js).

**Fallbacks (Socket.IO)** : Mécanismes de secours utilisés par Socket.IO pour établir ou maintenir une connexion lorsque la connexion WebSocket native n'est pas disponible ou échoue.

**Reconnexion automatique (Socket.IO)** : Fonctionnalité de Socket.IO qui tente de rétablir une connexion perdue sans intervention manuelle de l'utilisateur.

**Scalabilité horizontale** : Augmentation de la capacité d'un système en ajoutant davantage d'instances ou de serveurs, plutôt qu'en augmentant la puissance d'une seule machine.

**Brokers de messages** : Composant intermédiaire qui facilite la communication asynchrone entre différentes parties d'une application en gérant des files d'attente de messages ou en distribuant des événements (modèle Pub/Sub).

**Publish/Subscribe (Pub/Sub)** : Modèle de messagerie où les expéditeurs (publishers) envoient des messages à des sujets (topics) sans connaître les destinataires, et les récepteurs (subscribers) s'abonnent aux sujets qui les intéressent.

**Load balancer** : Dispositif ou logiciel qui distribue le trafic réseau entrant entre plusieurs instances de serveurs pour optimiser la performance et la fiabilité.

**Adaptateur Socket.IO (Redis)** : Module complémentaire de Socket.IO, comme celui basé sur Redis, qui permet à plusieurs instances de serveurs Socket.IO de communiquer entre elles et de diffuser des événements de manière cohérente.

**Bases de données NoSQL** : Catégorie de systèmes de gestion de bases de données qui offrent une plus grande flexibilité pour gérer des données non structurées ou semi-structurées, sans le modèle relationnel traditionnel.

**Bases de données en mémoire** : Bases de données qui stockent les données principalement dans la mémoire vive de l'ordinateur pour un accès ultra-rapide, optimisées pour la vitesse.

**Event sourcing** : Modèle de conception où l'état d'une application est déterminé par une séquence d'événements immuables, enregistrés dans un journal d'événements.

**Firebase Realtime Database** : Base de données NoSQL hébergée dans le cloud par Firebase, conçue pour synchroniser automatiquement les données entre les clients en temps réel.

**Firestore** : Base de données NoSQL flexible et scalable de Firebase, optimisée pour la synchronisation en temps réel et la gestion de données complexes, avec des capacités de requêtes avancées.

**Supabase Realtime** : Fonctionnalité de Supabase qui permet d'écouter les changements en temps réel dans une base de données PostgreSQL et de les diffuser aux clients.

**Webhooks** : Mécanisme permettant à une application d'envoyer des notifications automatiques (requêtes HTTP POST) à une autre application lorsqu'un événement spécifique se produit.

**CRDTs (Conflict-free Replicated Data Types)** : Structures de données conçues pour être répliquées et mises à jour indépendamment sur plusieurs systèmes tout en garantissant une convergence automatique vers un état cohérent.

**Sécurité des applications temps réel** : Ensemble de mesures visant à protéger les applications temps réel contre les menaces, incluant l'authentification, l'autorisation, la prévention des attaques DDoS et le chiffrement.

**Authentification** : Processus de vérification de l'identité d'un utilisateur ou d'un système.

**Autorisation** : Processus qui détermine les droits d'accès ou les actions qu'un utilisateur ou un système authentifié est autorisé à effectuer.

**Attaques DDoS** : Attaque visant à rendre un service indisponible en le submergeant d'un grand volume de trafic ou de requêtes non valides.

**Chiffrement** : Processus de transformation des informations pour les rendre illisibles à toute personne non autorisée, assurant la confidentialité des données.

**Gestion des identités et des sessions** : Processus de gestion des informations d'identification des utilisateurs et de leur état de connexion dans un environnement applicatif.

**Déploiement** : Processus de mise en production d'une application pour la rendre accessible aux utilisateurs sur une infrastructure cible.

**PaaS (Platform as a Service)** : Modèle de cloud computing qui fournit une plateforme et un environnement complets pour développer, exécuter et gérer des applications sans gérer l'infrastructure sous-jacente.

**Conteneurisation (Docker/Kubernetes)** : Méthode d'empaquetage d'une application et de toutes ses dépendances dans un conteneur isolé, facilitant son déploiement cohérent dans divers environnements.

**Monitoring** : Surveillance continue des performances, de la disponibilité, de la santé et des erreurs d'un système ou d'une application.

**Métriques** : Mesures quantifiables utilisées pour évaluer les performances, la santé et l'utilisation des ressources d'une application ou d'un système.

**Logs** : Enregistrements chronologiques des événements et des opérations d'un système, essentiels pour le débogage, le monitoring et l'audit.

**Traçabilité** : Capacité à suivre le parcours d'un événement ou d'une transaction à travers un système, utile pour le diagnostic et l'analyse.

**Optimisation des performances** : Processus d'amélioration de l'efficacité et de la vitesse d'une application pour garantir une expérience utilisateur fluide et une utilisation efficace des ressources.

**Compression** : Technique de réduction de la taille des données pour accélérer leur transmission et économiser la bande passante.

**Batching** : Regroupement de plusieurs opérations ou messages en un seul lot pour réduire le nombre de requêtes et améliorer l'efficacité.

**Throttling** : Limitation de la fréquence ou du nombre de requêtes ou d'événements qu'un système peut traiter dans un laps de temps donné pour éviter la surcharge.

**IoT (Internet des Objets)** : Réseau d'objets physiques (appareils, capteurs) connectés qui collectent et échangent des données en temps réel.

