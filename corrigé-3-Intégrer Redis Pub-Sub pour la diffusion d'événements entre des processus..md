Voici deux solutions complètes pour le TP sur la scalabilité d'une application de chat en temps réel avec Redis Pub/Sub. Chaque solution utilise une technologie back-end différente mais s'appuie sur le même principe de diffusion via Redis.

---

# Solution 1 : Scalabilité avec Node.js et Socket.IO

Cette solution utilise Node.js avec le framework Express pour le serveur web, la bibliothèque Socket.IO pour la communication en temps réel, et `ioredis` pour l'intégration de Redis Pub/Sub.

## Préparation de l'Environnement

1.  **Lancez Redis :**
    Assurez-vous qu'une instance Redis est en cours d'exécution. Le plus simple est avec Docker :
    
```bash
    docker run --name my-redis -p 6379:6379 -d redis/redis-stack-server:latest
    ```


2.  **Créez le projet Node.js :**
    
```bash
    mkdir chat-redis-node
    cd chat-redis-node
    npm init -y
    npm install express socket.io ioredis
    ```


3.  **Structure des fichiers :**
    
```
    chat-redis-node/
    ├── node_modules/
    ├── package.json
    ├── server.js
    └── public/
        └── index.html
    ```


## Partie 1 : Le Serveur (`server.js`)

Le serveur Node.js gérera les connexions Socket.IO et utilisera `ioredis` pour publier les messages entrants sur un canal Redis et s'abonner à ce même canal pour diffuser les messages aux clients locaux.

### Code du Serveur (`server.js`)




```javascript
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const path = require('path');
const Redis = require('ioredis'); // Importe la bibliothèque ioredis

const app = express();
const server = http.createServer(app);

// Initialise Socket.IO et le lie au serveur HTTP
const io = new Server(server);

// --- Configuration Redis ---
// Crée deux clients Redis : un pour la publication, un pour la souscription.
// C'est une bonne pratique car un client abonné ne peut pas effectuer d'autres commandes.
const redisPublisher = new Redis();
const redisSubscriber = new Redis();
const REDIS_CHANNEL = 'chat_messages'; // Nom du canal Redis pour les messages de chat

// Servir les fichiers statiques (comme index.html) depuis le dossier 'public'
app.use(express.static(path.join(__dirname, 'public')));

// Écoute des connexions Socket.IO
io.on('connection', (socket) => {
    console.log(`Un utilisateur est connecté (ID: ${socket.id})`);

    // Événement déclenché lorsqu'un client envoie un message de chat
    socket.on('chat message', (msg) => {
        console.log(`Message reçu du client ${socket.id}: ${msg}`);
        
        // Publie le message sur le canal Redis.
        // Le message est sérialisé en JSON pour conserver sa structure (expéditeur, contenu, etc.).
        redisPublisher.publish(REDIS_CHANNEL, JSON.stringify({
            senderId: socket.id, // ID du socket qui a envoyé le message
            message: msg,
            timestamp: Date.now()
        }));
    });

    // Événement déclenché lorsqu'un client se déconnecte
    socket.on('disconnect', () => {
        console.log(`Un utilisateur est déconnecté (ID: ${socket.id})`);
    });
});

// --- Logique de souscription Redis ---
// S'abonne au canal Redis pour recevoir les messages publiés par toutes les instances du serveur.
redisSubscriber.subscribe(REDIS_CHANNEL, (err, count) => {
    if (err) {
        console.error("Erreur lors de la souscription à Redis:", err);
        return;
    }
    console.log(`Souscrit au canal Redis '${REDIS_CHANNEL}' sur ${count} instance(s).`);
});

// Gère les messages reçus via Redis Pub/Sub
redisSubscriber.on('message', (channel, message) => {
    if (channel === REDIS_CHANNEL) {
        try {
            const parsedMessage = JSON.parse(message);
            console.log(`Message reçu de Redis sur le canal '${channel}':`, parsedMessage);

            // Diffuse le message à TOUS les clients connectés à cette instance de serveur.
            // On pourrait ajouter une logique pour ne pas renvoyer le message à l'expéditeur original
            // si l'on veut éviter un double affichage côté client, mais le client gère déjà cela.
            io.emit('chat message', parsedMessage.message); 

        } catch (e) {
            console.error("Erreur de parsing du message Redis:", e);
        }
    }
});

// Démarre le serveur HTTP sur le port spécifié (par défaut 3000)
const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
    console.log(`Serveur Socket.IO en écoute sur le port ${PORT}`);
});
```




### Explications du Serveur

*   **`ioredis` :** La bibliothèque `ioredis` est utilisée pour interagir avec Redis. Deux instances de client sont créées : `redisPublisher` et `redisSubscriber`. C'est une pratique courante car un client Redis abonné ne peut pas exécuter d'autres commandes.
*   **`socket.on('chat message', ...)` (Publication) :** Lorsqu'un message est reçu d'un client via Socket.IO, le serveur ne le diffuse pas directement. Au lieu de cela, il le sérialise en JSON (pour inclure des métadonnées comme l'ID de l'expéditeur et un timestamp) et le publie sur le `REDIS_CHANNEL` via `redisPublisher.publish()`.
*   **`redisSubscriber.subscribe(REDIS_CHANNEL, ...)` (Souscription) :** Au démarrage, chaque instance du serveur s'abonne au `REDIS_CHANNEL`. Cette opération est non bloquante avec `ioredis`.
*   **`redisSubscriber.on('message', ...)` (Diffusion) :** Lorsqu'un message est reçu de Redis (provenant de n'importe quelle instance de serveur), ce gestionnaire est déclenché.
    *   Le message est désérialisé (`JSON.parse()`).
    *   `io.emit('chat message', parsedMessage.message)` : Le message est ensuite diffusé à *tous les clients Socket.IO connectés à cette instance de serveur spécifique*. C'est ainsi que la cohérence est maintenue : chaque instance reçoit tous les messages via Redis et les transmet à ses propres clients.
*   **`process.env.PORT || 3000` :** Permet de lancer le serveur sur différents ports en utilisant la variable d'environnement `PORT`, ce qui est crucial pour les tests multi-instances.

## Partie 2 : Le Client (`public/index.html`)

Le client est une page HTML simple avec du JavaScript qui utilise la bibliothèque client Socket.IO pour se connecter au serveur et afficher les messages. Le code client reste largement inchangé par rapport à une application de chat Socket.IO de base.

### Code du Client (`public/index.html`)




```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chat Scalable (Node.js/Redis)</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 20px; background-color: #f4f4f4; }
        #chat-container { max-width: 700px; margin: 20px auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h1 { text-align: center; color: #333; margin-bottom: 20px; }
        #messages { list-style-type: none; margin: 0; padding: 0; max-height: 400px; overflow-y: auto; border: 1px solid #eee; padding: 10px; margin-bottom: 10px; background-color: #fdfdfd; border-radius: 4px; }
        #messages li { padding: 8px 10px; border-bottom: 1px solid #eee; }
        #messages li:last-child { border-bottom: none; }
        #messages li strong { color: #007bff; }
        #form-chat { display: flex; }
        #message-input { flex-grow: 1; padding: 10px; border: 1px solid #ddd; border-radius: 4px; margin-right: 5px; }
        #send-button { padding: 10px 15px; background-color: #28a745; color: white; border: none; border-radius: 4px; cursor: pointer; }
        #send-button:hover { background-color: #218838; }
        .status-message { font-style: italic; color: #666; text-align: center; }
    </style>
</head>
<body>
    <div id="chat-container">
        <h1>Chat Scalable</h1>
        <ul id="messages"></ul>
        <form id="form-chat">
            <input id="message-input" autocomplete="off" placeholder="Votre message..." />
            <button id="send-button" type="submit">Envoyer</button>
        </form>
        <p id="connection-status" class="status-message">Connexion en cours...</p>
    </div>

    <!-- La bibliothèque client Socket.IO est servie automatiquement par le serveur Socket.IO -->
    <script src="/socket.io/socket.io.js"></script>
    <script>
        const socket = io(); // Initialise la connexion Socket.IO

        const messages = document.getElementById('messages');
        const formChat = document.getElementById('form-chat');
        const messageInput = document.getElementById('message-input');
        const connectionStatus = document.getElementById('connection-status');

        // Fonction pour ajouter un message à la liste
        function addMessageToChat(msg, isStatus = false) {
            const item = document.createElement('li');
            if (isStatus) {
                item.classList.add('status-message');
                item.textContent = msg;
            } else {
                item.textContent = msg;
            }
            messages.appendChild(item);
            messages.scrollTop = messages.scrollHeight; // Scroll automatique
        }

        // Gère la soumission du formulaire de chat
        formChat.addEventListener('submit', (e) => {
            e.preventDefault(); // Empêche le rechargement de la page
            if (messageInput.value) {
                socket.emit('chat message', messageInput.value); // Envoie le message au serveur
                addMessageToChat(`Moi : ${messageInput.value}`); // Affiche le message localement
                messageInput.value = ''; // Vide le champ de saisie
            }
        });

        // Écoute les messages entrants du serveur
        socket.on('chat message', (msg) => {
            // Pour éviter le double affichage si le message vient de soi-même
            // (le serveur renvoie tous les messages, y compris ceux que l'on a envoyés)
            // Une logique plus fine pourrait être d'inclure l'ID de l'expéditeur dans le message
            // et de ne pas afficher si senderId === socket.id.
            // Pour ce TP, on se contente d'afficher tous les messages reçus du serveur.
            addMessageToChat(`Autre : ${msg}`);
        });

        // Gère les événements de connexion/déconnexion Socket.IO
        socket.on('connect', () => {
            console.log('Connecté au serveur Socket.IO');
            connectionStatus.textContent = 'Statut : Connecté';
            connectionStatus.style.color = 'green';
        });

        socket.on('disconnect', () => {
            console.log('Déconnecté du serveur Socket.IO');
            connectionStatus.textContent = 'Statut : Déconnecté';
            connectionStatus.style.color = 'red';
        });

        socket.on('connect_error', (error) => {
            console.error('Erreur de connexion Socket.IO:', error);
            connectionStatus.textContent = 'Statut : Erreur de connexion';
            connectionStatus.style.color = 'red';
        });
    </script>
</body>
</html>
```




### Explications du Client

*   Le client est une application Socket.IO standard. Il se connecte à une instance de serveur, envoie des messages via `socket.emit('chat message', ...)` et écoute les messages entrants via `socket.on('chat message', ...)`.
*   Il n'a pas besoin de savoir que Redis est utilisé en arrière-plan. Il interagit uniquement avec l'instance de serveur à laquelle il est connecté.
*   Le message est affiché localement immédiatement après l'envoi pour une meilleure expérience utilisateur.

## Lancement et Test Multi-Instances

1.  **Lancez Redis** (si ce n'est pas déjà fait) :
    
```bash
    docker run --name my-redis -p 6379:6379 -d redis/redis-stack-server:latest
    ```


2.  **Lancez la première instance du serveur :**
    
```bash
    node server.js
    ```

    (Ceci lancera le serveur sur le port 3000 par défaut).

3.  **Lancez la deuxième instance du serveur (sur un port différent) :**
    Ouvrez un *nouveau terminal* dans le même dossier `chat-redis-node` et exécutez :
    
```bash
    PORT=3001 node server.js
    ```

    (Ceci lancera le serveur sur le port 3001). Vous pouvez lancer d'autres instances sur d'autres ports si vous le souhaitez.

4.  **Ouvrez les clients :**
    *   Ouvrez un navigateur et allez sur `http://localhost:3000`.
    *   Ouvrez un deuxième onglet (ou une autre fenêtre de navigateur) et allez sur `http://localhost:3001`.

5.  **Testez la communication :**
    Envoyez un message depuis l'onglet `localhost:3000`. Vous devriez voir ce message apparaître dans l'onglet `localhost:3001`, et vice-versa. Cela démontre que Redis Pub/Sub permet aux messages de traverser les différentes instances de serveur.

---

# Solution 2 : Scalabilité avec Python et Flask-SocketIO

Cette solution utilise Python avec le framework Flask pour le serveur web, Flask-SocketIO pour la communication en temps réel, et `redis-py` pour l'intégration de Redis Pub/Sub. Flask-SocketIO offre une intégration native et simplifiée de Redis pour la scalabilité.

## Préparation de l'Environnement

1.  **Lancez Redis :**
    Assurez-vous qu'une instance Redis est en cours d'exécution (même commande Docker que précédemment) :
    
```bash
    docker run --name my-redis -p 6379:6379 -d redis/redis-stack-server:latest
    ```


2.  **Créez le projet Python :**
    
```bash
    mkdir chat-redis-python
    cd chat-redis-python
    python -m venv venv
    source venv/bin/activate # Sur Windows: venv\Scripts\activate
    pip install Flask Flask-SocketIO redis
    ```


3.  **Structure des fichiers :**
    
```
    chat-redis-python/
    ├── venv/
    ├── app.py
    └── templates/
        └── index.html
    ```


## Partie 1 : Le Serveur (`app.py`)

Le serveur Flask gérera les requêtes HTTP et les connexions Socket.IO. Flask-SocketIO simplifie grandement l'intégration de Redis Pub/Sub en gérant la logique de publication et de souscription en arrière-plan via son paramètre `message_queue`.

### Code du Serveur (`app.py`)




```python
from flask import Flask, render_template, request
from flask_socketio import SocketIO, emit
import os

app = Flask(__name__)

# --- Configuration Flask-SocketIO avec Redis ---
# Le paramètre 'message_queue' indique à Flask-SocketIO d'utiliser Redis
# comme backend pour la diffusion des messages entre les instances de serveur.
# Flask-SocketIO gérera automatiquement la publication et la souscription.
app.config['SECRET_KEY'] = 'your_secret_key' # Clé secrète requise pour Flask-SocketIO
app.config['SOCKETIO_MESSAGE_QUEUE'] = 'redis://localhost:6379/0' # URL de votre instance Redis

# Initialise Socket.IO avec l'application Flask et la file de messages Redis
socketio = SocketIO(app, message_queue=app.config['SOCKETIO_MESSAGE_QUEUE'], async_mode='eventlet')

# Route pour servir la page HTML du client
@app.route('/')
def index():
    return render_template('index.html')

# --- Gestion des événements Socket.IO ---

@socketio.on('connect')
def handle_connect():
    print(f'Client connecté: {request.sid}')
    # Optionnel: émettre un message de statut au client
    # emit('status', {'msg': 'Connecté au chat !'})

@socketio.on('disconnect')
def handle_disconnect():
    print(f'Client déconnecté: {request.sid}')

# Événement déclenché lorsqu'un client envoie un message de chat
@socketio.on('chat message')
def handle_chat_message(data):
    print(f'Message reçu du client {request.sid}: {data}')
    
    # Diffuse le message à TOUS les clients connectés,
    # y compris ceux connectés à d'autres instances de serveur via Redis.
    # Flask-SocketIO s'occupe de publier sur Redis et de le rediffuser.
    emit('chat message', data, broadcast=True)

if __name__ == '__main__':
    # Pour lancer plusieurs instances sur des ports différents,
    # on peut utiliser la variable d'environnement FLASK_RUN_PORT
    # ou passer le port directement à socketio.run().
    # Par exemple: socketio.run(app, port=5000)
    # Pour le développement, debug=True est utile.
    socketio.run(app, debug=True, port=5000)
```




### Explications du Serveur

*   **`app.config['SOCKETIO_MESSAGE_QUEUE'] = 'redis://localhost:6379/0'` :** C'est la configuration clé. En spécifiant une URL Redis ici, Flask-SocketIO est instruit d'utiliser Redis comme backend pour sa file de messages. Cela signifie que toutes les opérations de diffusion (`emit(..., broadcast=True)`) seront gérées via Redis Pub/Sub en arrière-plan.
*   **`socketio = SocketIO(app, message_queue=..., async_mode='eventlet')` :** Initialise l'extension Flask-SocketIO avec la configuration Redis. `async_mode='eventlet'` est souvent recommandé pour Flask-SocketIO afin de gérer efficacement les connexions concurrentes et les opérations I/O non bloquantes, y compris Redis.
*   **`@socketio.on('chat message')` :**
    *   Lorsqu'un message est reçu d'un client, la fonction `handle_chat_message` est appelée.
    *   `emit('chat message', data, broadcast=True)` : Cette ligne est le cœur de la diffusion. Grâce à la configuration `message_queue` avec Redis, `broadcast=True` ne signifie plus seulement "diffuser à tous les clients de cette instance", mais "diffuser à tous les clients connectés à *n'importe quelle* instance de serveur via Redis". Flask-SocketIO gère la publication sur Redis et la souscription/rediffusion sur les autres instances automatiquement.
*   **`socketio.run(app, debug=True, port=5000)` :** Démarre le serveur. Pour lancer plusieurs instances sur des ports différents, vous devrez modifier le port ici ou utiliser des variables d'environnement.

## Partie 2 : Le Client (`templates/index.html`)

Le client est une page HTML simple avec du JavaScript qui utilise la bibliothèque client Socket.IO. Le code client est identique à celui de la solution Node.js, car l'API Socket.IO côté client est standard.

### Code du Client (`templates/index.html`)




```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chat Scalable (Python/Redis)</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 20px; background-color: #f4f4f4; }
        #chat-container { max-width: 700px; margin: 20px auto; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h1 { text-align: center; color: #333; margin-bottom: 20px; }
        #messages { list-style-type: none; margin: 0; padding: 0; max-height: 400px; overflow-y: auto; border: 1px solid #eee; padding: 10px; margin-bottom: 10px; background-color: #fdfdfd; border-radius: 4px; }
        #messages li { padding: 8px 10px; border-bottom: 1px solid #eee; }
        #messages li:last-child { border-bottom: none; }
        #messages li strong { color: #007bff; }
        #form-chat { display: flex; }
        #message-input { flex-grow: 1; padding: 10px; border: 1px solid #ddd; border-radius: 4px; margin-right: 5px; }
        #send-button { padding: 10px 15px; background-color: #28a745; color: white; border: none; border-radius: 4px; cursor: pointer; }
        #send-button:hover { background-color: #218838; }
        .status-message { font-style: italic; color: #666; text-align: center; }
    </style>
</head>
<body>
    <div id="chat-container">
        <h1>Chat Scalable</h1>
        <ul id="messages"></ul>
        <form id="form-chat">
            <input id="message-input" autocomplete="off" placeholder="Votre message..." />
            <button id="send-button" type="submit">Envoyer</button>
        </form>
        <p id="connection-status" class="status-message">Connexion en cours...</p>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.0.1/socket.io.min.js"></script>
    <script>
        const socket = io(); // Initialise la connexion Socket.IO

        const messages = document.getElementById('messages');
        const formChat = document.getElementById('form-chat');
        const messageInput = document.getElementById('message-input');
        const connectionStatus = document.getElementById('connection-status');

        function addMessageToChat(msg, isStatus = false) {
            const item = document.createElement('li');
            if (isStatus) {
                item.classList.add('status-message');
                item.textContent = msg;
            } else {
                item.textContent = msg;
            }
            messages.appendChild(item);
            messages.scrollTop = messages.scrollHeight;
        }

        formChat.addEventListener('submit', (e) => {
            e.preventDefault();
            if (messageInput.value) {
                socket.emit('chat message', messageInput.value);
                addMessageToChat(`Moi : ${messageInput.value}`);
                messageInput.value = '';
            }
        });

        socket.on('chat message', (msg) => {
            addMessageToChat(`Autre : ${msg}`);
        });

        socket.on('connect', () => {
            console.log('Connecté au serveur Socket.IO');
            connectionStatus.textContent = 'Statut : Connecté';
            connectionStatus.style.color = 'green';
        });

        socket.on('disconnect', () => {
            console.log('Déconnecté du serveur Socket.IO');
            connectionStatus.textContent = 'Statut : Déconnecté';
            connectionStatus.style.color = 'red';
        });

        socket.on('connect_error', (error) => {
            console.error('Erreur de connexion Socket.IO:', error);
            connectionStatus.textContent = 'Statut : Erreur de connexion';
            connectionStatus.style.color = 'red';
        });
    </script>
</body>
</html>
```




### Explications du Client

*   Le client est une application Socket.IO standard. Il se connecte à une instance de serveur, envoie des messages via `socket.emit('chat message', ...)` et écoute les messages entrants via `socket.on('chat message', ...)`.
*   Il n'a pas besoin de savoir que Redis est utilisé en arrière-plan. Il interagit uniquement avec l'instance de serveur à laquelle il est connecté.
*   Le message est affiché localement immédiatement après l'envoi pour une meilleure expérience utilisateur.
*   Notez l'utilisation de `https://cdnjs.cloudflare.com/ajax/libs/socket.io/4.0.1/socket.io.min.js` pour inclure la bibliothèque client Socket.IO, car Flask ne la sert pas automatiquement comme Express.

## Lancement et Test Multi-Instances

1.  **Lancez Redis** (si ce n'est pas déjà fait) :
    
```bash
    docker run --name my-redis -p 6379:6379 -d redis/redis-stack-server:latest
    ```


2.  **Lancez la première instance du serveur :**
    Assurez-vous d'être dans votre environnement virtuel (`source venv/bin/activate`).
    
```bash
    python app.py
    ```

    (Ceci lancera le serveur sur le port 5000 par défaut).

3.  **Lancez la deuxième instance du serveur (sur un port différent) :**
    Ouvrez un *nouveau terminal*, activez votre environnement virtuel, et exécutez :
    
```bash
    FLASK_APP=app.py FLASK_RUN_PORT=5001 flask run
    ```

    (Ceci lancera le serveur sur le port 5001). Vous pouvez lancer d'autres instances sur d'autres ports si vous le souhaitez.

4.  **Ouvrez les clients :**
    *   Ouvrez un navigateur et allez sur `http://localhost:5000`.
    *   Ouvrez un deuxième onglet (ou une autre fenêtre de navigateur) et allez sur `http://localhost:5001`.

5.  **Testez la communication :**
    Envoyez un message depuis l'onglet `localhost:5000`. Vous devriez voir ce message apparaître dans l'onglet `localhost:5001`, et vice-versa. Cela démontre que Redis Pub/Sub permet aux messages de traverser les différentes instances de serveur, géré de manière transparente par Flask-SocketIO.

---

Ces deux solutions illustrent comment Redis Pub/Sub peut être intégré pour rendre une application de chat scalable. La solution Flask-SocketIO est particulièrement élégante car elle abstrait une grande partie de la logique Pub/Sub, tandis que la solution Node.js montre comment implémenter cette logique manuellement avec `ioredis`.