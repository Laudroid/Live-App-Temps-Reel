# Concepts clés de Socket.IO : Événements, namespaces, rooms

## Introduction

Socket.IO introduit plusieurs concepts pour structurer efficacement la communication temps réel entre clients et serveur : la gestion d'événements, les namespaces, et les rooms. Ces notions facilitent la création d’applications complexes, avec un contrôle fin sur les flux de données et la segmentation des utilisateurs.

---

## 1. Événements : base de la communication

Socket.IO repose sur un modèle **orienté événements**, qui permet d’envoyer et recevoir des messages nommés à travers la connexion.

- Le serveur et le client écoutent puis émettent des événements avec un nom personnalisé (ex : `'message'`, `'join'`, `'update'`).
- Chaque événement peut transporter des données (objets JSON, chaînes, etc.).

### Exemple simple d’envoi et réception d’événements

**Côté client :**

```javascript
socket.on('message', (data) => {
    console.log('Message reçu :', data);
});

socket.emit('message', 'Bonjour serveur');
```

**Côté serveur :**

```javascript
io.on('connection', (socket) => {
    socket.on('message', (msg) => {
        console.log('Message client:', msg);
        socket.emit('message', 'Message reçu');
    });
});
```

---

## 2. Namespaces : isolation logique pour plusieurs canaux

### Définition

Un **namespace** est un canal de communication distinct identifié par une URL (commençant par `/`), permettant d’isoler différentes parties d’une application au sein du même serveur Socket.IO.

- Par défaut, tous les clients se connectent au namespace `/` (racine).
- On peut créer des namespaces dédiés (`/chat`, `/news`) pour segmenter les communications.

### Utilité

- Séparer les responsabilités logiques.
- Optimiser les ressources en ne diffusant pas des messages inutiles aux clients non concernés.

### Exemple de création d’un namespace sur serveur

```javascript
const chatNamespace = io.of('/chat');

chatNamespace.on('connection', (socket) => {
    console.log('Client connecté au namespace /chat');
    socket.on('message', (msg) => {
        chatNamespace.emit('message', msg); // Diffuse à tous dans /chat
    });
});
```

**Côté client, la connexion :**

```javascript
const chatSocket = io('/chat');
chatSocket.emit('message', "Salut depuis /chat");
```

---

## 3. Rooms : sous-groupes dans un namespace

### Définition

Une **room** est un sous-groupe de sockets au sein d’un namespace. Elle permet de diffuser des messages ciblés à un ensemble limité de clients.

- Un socket peut rejoindre plusieurs rooms.
- Permet un broadcasting précis, par exemple uniquement aux utilisateurs dans une même salle de chat.

### Exemple de gestion de room

```javascript
io.on('connection', (socket) => {
    socket.join('room1'); // Ajout à la room "room1"

    socket.to('room1').emit('message', 'Un nouveau membre a rejoint la room');

    socket.on('message', (msg) => {
        // Envoi du message uniquement aux autres membres de la room "room1"
        socket.to('room1').emit('message', msg);
    });
});
```

---

## 4. Diagramme des interactions namespaces/rooms

```mermaid
graph TD
    Client1((Client))
    Client2((Client))
    Client3((Client))

    subgraph Namespace1 /chat
        Room1((Room : "room1"))
        Room2((Room : "room2"))
    end

    Client1 -->|connect| Namespace1
    Client2 -->|connect| Namespace1
    Client3 -->|connect| Namespace1

    Client1 -->|join| Room1
    Client2 -->|join| Room1
    Client3 -->|join| Room2

    Room1 -->|broadcast msg| Client1 & Client2
    Room2 -->|broadcast msg| Client3
```

---

## 5. Sources

- Socket.IO Documentation – [Namespaces](https://socket.io/docs/v4/namespaces/)  
- Socket.IO Documentation – [Rooms](https://socket.io/docs/v4/rooms/)  
- MDN Web Docs – [Using WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)  
- GitHub socket.io examples : https://github.com/socketio/socket.io/tree/main/examples  

---

Socket.IO structure la communication avec un système d’événements souple et des mécanismes puissants de segmentation (namespaces, rooms). Ceci permet d’adapter la diffusion des messages en fonction des groupes d’utilisateurs et des fonctionnalités, assurant un contrôle précis et efficace des échanges en temps réel.