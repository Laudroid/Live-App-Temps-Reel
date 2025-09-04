# Diffusion de messages avec Socket.IO : broadcast, to, emit

## Introduction

Socket.IO propose plusieurs méthodes pour diffuser des messages entre clients connectés à un serveur, permettant de choisir précisément la portée de chaque communication. Les principaux modes sont `emit`, `broadcast` et `to` (alias `in`), chacun ayant un comportement distinct pour l’envoi des messages.

---

## 1. `emit` : envoyer un message ciblé

La méthode `emit` envoie un événement avec des données à un socket précis, généralement celui qui est connecté ou un objet socket particulier.

- **Côté serveur**, utiliser `socket.emit` pour répondre uniquement au client initiateur.
- **Côté client**, utiliser `socket.emit` pour envoyer un message au serveur.

**Exemple serveur :**

```javascript
io.on('connection', (socket) => {
    socket.emit('welcome', 'Bienvenue sur le serveur');
});
```

**Exemple client :**

```javascript
socket.emit('myEvent', { data: 'message au serveur' });
```

---

## 2. `broadcast` : diffusion à tous sauf émetteur

Le mode `broadcast` permet d’envoyer un message à **tous les clients connectés sauf celui qui envoie**.

**Exemple d’utilisation serveur :**

```javascript
io.on('connection', (socket) => {
    socket.on('message', (msg) => {
        socket.broadcast.emit('message', msg);
    });
});
```

Dans cet exemple, quand un client envoie un message, tous les autres reçoivent cet événement, mais pas l’émetteur.

---

## 3. `to` ou `in` : diffusion ciblée à une room ou un namespace

Les méthodes `to()` et `in()` limitent la diffusion à une room spécifique. Elles s’utilisent souvent en combinaison avec `emit()`.

**Exemple : Diffuser à une room nommée `room1`**

```javascript
io.on('connection', (socket) => {
    socket.join('room1');
    socket.to('room1').emit('message', 'Hello room1!');
});
```

- En utilisant `socket.to('room1').emit()`, le message est envoyé à tous les sockets membres de la room `room1`, **sauf** le socket émetteur.
- Pour envoyer à **tous** les clients dans une room, y compris l’émetteur, utiliser `io.in('room1').emit()`.

---

## 4. Synthèse des modes d’émission

| Fonction                 | Cible                                  | Inclus l’émetteur ? | Exemple                                     |
|--------------------------|---------------------------------------|---------------------|---------------------------------------------|
| `socket.emit(event, data)` | Socket spécifique (généralement émetteur) | Oui                 | Réponse au client                             |
| `socket.broadcast.emit(event, data)` | Tous les sockets sauf émetteur          | Non                 | Diffusion à tous sauf celui qui envoie       |
| `socket.to(room).emit(event, data)` | Tous les sockets dans la room sauf émetteur | Non                 | Message aux membres d’une room (hors émetteur) |
| `io.in(room).emit(event, data)` | Tous les sockets dans la room               | Oui                 | Message à tous les membres d’une room         |

---

## 5. Diagramme Mermaid illustrant les diffusions

```mermaid
graph LR
    A[Serveur Socket] 
    B1[Client 1 (Émetteur)]
    B2[Client 2]
    B3[Client 3]

    A -->|socket.emit| B1
    A -->|socket.broadcast.emit| B2 & B3
    A -->|socket.to('room').emit| B2
    A -->|io.in('room').emit| B2 & B1
```

---

## Sources

- Socket.IO Documentation – [Emitting Events](https://socket.io/docs/v4/emitting-events/)  
- Socket.IO Guide – [Rooms and Namespaces](https://socket.io/docs/v4/rooms/)  
- MDN Web Docs – [WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)  

---

Socket.IO offre une flexibilité complète dans la diffusion des messages grâce à `emit`, `broadcast` et `to/in`, permettant de cibler avec précision les récepteurs des événements. Cette granularité est essentielle pour optimiser la gestion des échanges dans des applications temps réel.