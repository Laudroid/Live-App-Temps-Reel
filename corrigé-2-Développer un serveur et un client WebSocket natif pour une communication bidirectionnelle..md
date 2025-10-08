Voici deux solutions pour le TP sur la communication bidirectionnelle avec WebSockets natifs. Elles sont conçues pour être claires et fonctionnelles, en mettant l'accent sur les mécanismes fondamentaux.

---

# Solution 1 : Implémentation Basique

Cette solution fournit une implémentation directe du serveur et du client WebSockets, répondant aux exigences minimales du TP.

## Partie 1 : Le Serveur WebSocket (Node.js)

Le serveur utilisera la bibliothèque `ws` pour gérer les connexions et le traitement des messages.

### Structure du Projet



```
tp-websocket-capitalizer/
├── node_modules/
├── package.json
├── package-lock.json
├── server.js
└── public/
    ├── index.html
    └── client.js
```



### Code du Serveur (`server.js`)



```javascript
const WebSocket = require('ws'); // Importe la classe WebSocket.Server de la bibliothèque 'ws'

// Crée une instance de serveur WebSocket écoutant sur le port 8080
const wss = new WebSocket.Server({ port: 8080 });

console.log('Serveur WebSocket démarré et écoute sur le port 8080');

// Événement déclenché lorsqu'un nouveau client établit une connexion
wss.on('connection', ws => {
    console.log('Nouveau client connecté !');

    // Événement déclenché lorsqu'un message est reçu de ce client spécifique
    ws.on('message', message => {
        // Les messages WebSocket peuvent être des Buffers (binaires) ou des chaînes de caractères.
        // On s'assure de le convertir en chaîne pour le traitement.
        const receivedMessage = message.toString('utf8'); 
        console.log(`Message reçu du client : "${receivedMessage}"`);

        // Capitalise la chaîne reçue
        const capitalizedMessage = receivedMessage.toUpperCase();
        console.log(`Message capitalisé : "${capitalizedMessage}"`);

        // Renvoye la chaîne capitalisée au client qui a envoyé le message
        ws.send(capitalizedMessage);
    });

    // Événement déclenché lorsque la connexion de ce client est fermée
    ws.on('close', () => {
        console.log('Client déconnecté.');
    });

    // Événement déclenché en cas d'erreur sur cette connexion
    ws.on('error', error => {
        console.error('Erreur WebSocket pour le client:', error);
    });

    // Envoie un message initial au client pour confirmer la connexion (optionnel)
    ws.send('Connecté au serveur de capitalisation !');
});
```



### Explications du Serveur

*   **`new WebSocket.Server({ port: 8080 })` :** Initialise le serveur WebSocket. Il écoute les tentatives de connexion sur le port spécifié.
*   **`wss.on('connection', ws => { ... })` :** C'est le point d'entrée pour chaque nouveau client. L'objet `ws` représente la connexion individuelle avec ce client.
*   **`ws.on('message', message => { ... })` :** Ce gestionnaire est appelé chaque fois que le client envoie des données.
    *   `message.toString('utf8')` : Convertit les données brutes (Buffer) en une chaîne de caractères.
    *   `receivedMessage.toUpperCase()` : Effectue la capitalisation.
    *   `ws.send(capitalizedMessage)` : Renvoye la chaîne traitée *au même client* qui l'a envoyée.
*   **`ws.on('close', ...)` et `ws.on('error', ...)` :** Ces gestionnaires sont essentiels pour surveiller l'état des connexions et diagnostiquer les problèmes.

## Partie 2 : Le Client WebSocket (HTML/JavaScript)

Le client sera une page HTML simple avec du JavaScript natif pour établir la connexion et interagir avec le serveur.

### Code HTML (`public/index.html`)



```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Client WebSocket Capitalizer</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background-color: #f4f4f4; }
        .container {
            max-width: 500px;
            margin: 0 auto;
            background-color: #fff;
            padding: 25px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 { text-align: center; color: #333; margin-bottom: 25px; }
        input[type="text"] {
            width: calc(100% - 100px); /* Ajuste la largeur pour le bouton */
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            margin-right: 10px;
            box-sizing: border-box;
        }
        button {
            padding: 10px 15px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
        #responseDisplay {
            margin-top: 20px;
            padding: 15px;
            border: 1px solid #e0e0e0;
            background-color: #e9e9e9;
            border-radius: 4px;
            min-height: 50px;
            font-size: 1.1em;
            color: #333;
            word-wrap: break-word; /* Gère les longs mots */
        }
        .status-message {
            font-style: italic;
            color: #6c757d;
            margin-top: 10px;
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Capitaliseur WebSocket</h1>
        <input type="text" id="messageInput" placeholder="Entrez du texte ici...">
        <button id="sendButton">Envoyer</button>
        <div id="responseDisplay">
            <!-- La réponse du serveur s'affichera ici -->
        </div>
        <p class="status-message" id="connectionStatus">Connexion en cours...</p>
    </div>

    <script src="client.js" defer></script>
</body>
</html>
```



### Code JavaScript (`public/client.js`)



```javascript
document.addEventListener('DOMContentLoaded', () => {
    const messageInput = document.getElementById('messageInput');
    const sendButton = document.getElementById('sendButton');
    const responseDisplay = document.getElementById('responseDisplay');
    const connectionStatus = document.getElementById('connectionStatus');

    // Crée une nouvelle instance de WebSocket pour se connecter au serveur
    // L'URL doit correspondre à l'adresse et au port du serveur WebSocket.
    const socket = new WebSocket('ws://localhost:8080');

    // --- Gestion des événements WebSocket ---

    // Événement déclenché lorsque la connexion est établie avec succès
    socket.onopen = function(event) {
        console.log('Connecté au serveur WebSocket.');
        connectionStatus.textContent = 'Statut : Connecté';
        connectionStatus.style.color = 'green';
    };

    // Événement déclenché lorsqu'un message est reçu du serveur
    socket.onmessage = function(event) {
        console.log('Message reçu du serveur:', event.data);
        responseDisplay.textContent = `Réponse du serveur : ${event.data}`;
    };

    // Événement déclenché en cas d'erreur de connexion ou de communication
    socket.onerror = function(error) {
        console.error('Erreur WebSocket:', error);
        connectionStatus.textContent = 'Statut : Erreur de connexion';
        connectionStatus.style.color = 'red';
        responseDisplay.textContent = 'Une erreur est survenue lors de la communication.';
    };

    // Événement déclenché lorsque la connexion est fermée
    socket.onclose = function(event) {
        console.log('Déconnecté du serveur WebSocket.');
        connectionStatus.textContent = 'Statut : Déconnecté';
        connectionStatus.style.color = 'orange';
        responseDisplay.textContent = 'Connexion au serveur fermée.';
    };

    // --- Gestion de l'interface utilisateur ---

    // Écouteur d'événement pour le bouton d'envoi
    sendButton.addEventListener('click', () => {
        const message = messageInput.value;
        if (message.trim() !== '') {
            // Vérifie si la connexion est ouverte avant d'envoyer
            if (socket.readyState === WebSocket.OPEN) {
                socket.send(message); // Envoie le message au serveur
                messageInput.value = ''; // Vide le champ de saisie
                responseDisplay.textContent = 'Envoi en cours...';
            } else {
                console.warn('La connexion WebSocket n\'est pas ouverte.');
                responseDisplay.textContent = 'Impossible d\'envoyer : connexion non ouverte.';
            }
        }
    });

    // Permet d'envoyer le message en appuyant sur la touche "Entrée"
    messageInput.addEventListener('keypress', (event) => {
        if (event.key === 'Enter') {
            sendButton.click(); // Simule un clic sur le bouton
        }
    });
});
```



### Explications du Client

*   **`new WebSocket('ws://localhost:8080')` :** Crée une nouvelle instance de l'API `WebSocket` du navigateur. L'URL doit correspondre à celle de votre serveur.
*   **`socket.onopen` :** Se déclenche lorsque la connexion est établie. Le client peut alors commencer à envoyer des messages.
*   **`socket.onmessage = function(event) { ... }` :** Ce gestionnaire est appelé chaque fois que le serveur envoie un message. `event.data` contient la chaîne capitalisée, qui est ensuite affichée dans le `responseDisplay`.
*   **`socket.onerror` et `socket.onclose` :** Gèrent les erreurs et la fermeture de la connexion, fournissant un feedback visuel à l'utilisateur.
*   **`sendButton.addEventListener('click', ...)` :** Lorsque le bouton est cliqué, la valeur de l'input est envoyée au serveur via `socket.send()`. Une vérification `socket.readyState === WebSocket.OPEN` est ajoutée pour s'assurer que la connexion est active avant d'envoyer.
*   **`messageInput.addEventListener('keypress', ...)` :** Améliore l'ergonomie en permettant d'envoyer le message avec la touche "Entrée".

---

# Solution 2 : Implémentation avec Suivi des Messages Envoyés

Cette solution est similaire à la première mais ajoute une petite amélioration côté client pour afficher à la fois le message envoyé et la réponse du serveur, offrant un meilleur suivi de la communication.

## Partie 1 : Le Serveur WebSocket (Node.js)

Le code du serveur reste **identique** à la Solution 1, car sa logique de réception, capitalisation et renvoi est déjà conforme aux exigences.

### Code du Serveur (`server.js`)

(Le contenu est le même que `server.js` de la Solution 1)


```javascript
const WebSocket = require('ws'); 

const wss = new WebSocket.Server({ port: 8080 });

console.log('Serveur WebSocket démarré et écoute sur le port 8080');

wss.on('connection', ws => {
    console.log('Nouveau client connecté !');

    ws.on('message', message => {
        const receivedMessage = message.toString('utf8'); 
        console.log(`Message reçu du client : "${receivedMessage}"`);

        const capitalizedMessage = receivedMessage.toUpperCase();
        console.log(`Message capitalisé : "${capitalizedMessage}"`);

        ws.send(capitalizedMessage);
    });

    ws.on('close', () => {
        console.log('Client déconnecté.');
    });

    ws.on('error', error => {
        console.error('Erreur WebSocket pour le client:', error);
    });

    ws.send('Connecté au serveur de capitalisation !');
});
```


## Partie 2 : Le Client WebSocket (HTML/JavaScript)

Le client sera modifié pour afficher une trace des messages envoyés et des réponses reçues.

### Code HTML (`public/index.html`)

Le HTML est légèrement modifié pour avoir une zone d'affichage plus générique pour les messages.


```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Client WebSocket Capitalizer (Amélioré)</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 20px; background-color: #eef; }
        .container {
            max-width: 600px;
            margin: 20px auto;
            background-color: #fff;
            padding: 25px;
            border-radius: 10px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        h1 { text-align: center; color: #2c3e50; margin-bottom: 25px; }
        .input-area {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        input[type="text"] {
            flex-grow: 1;
            padding: 12px;
            border: 1px solid #ccc;
            border-radius: 5px;
            box-sizing: border-box;
            font-size: 1em;
        }
        button {
            padding: 12px 20px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 1em;
        }
        button:hover {
            background-color: #0056b3;
        }
        #messagesDisplay {
            margin-top: 20px;
            padding: 15px;
            border: 1px solid #e0e0e0;
            background-color: #f9f9f9;
            border-radius: 5px;
            min-height: 150px;
            max-height: 300px;
            overflow-y: auto;
            font-size: 0.95em;
            line-height: 1.6;
            color: #333;
        }
        .message-sent { color: #0056b3; font-weight: bold; }
        .message-received { color: #28a745; font-weight: bold; }
        .status-message { font-style: italic; color: #6c757d; text-align: center; margin: 10px 0; }
        .error-message { color: red; font-weight: bold; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Capitaliseur WebSocket</h1>
        <div class="input-area">
            <input type="text" id="messageInput" placeholder="Entrez du texte ici...">
            <button id="sendButton">Envoyer</button>
        </div>
        <div id="messagesDisplay">
            <!-- Les messages envoyés et reçus s'afficheront ici -->
        </div>
        <p class="status-message" id="connectionStatus">Connexion en cours...</p>
    </div>

    <script src="client.js" defer></script>
</body>
</html>
```



### Code JavaScript (`public/client.js`)



```javascript
document.addEventListener('DOMContentLoaded', () => {
    const messageInput = document.getElementById('messageInput');
    const sendButton = document.getElementById('sendButton');
    const messagesDisplay = document.getElementById('messagesDisplay'); // Nouvelle zone d'affichage
    const connectionStatus = document.getElementById('connectionStatus');

    const socket = new WebSocket('ws://localhost:8080');

    // Fonction utilitaire pour ajouter des messages à la zone d'affichage
    function addMessageToDisplay(message, type = 'status') {
        const p = document.createElement('p');
        p.textContent = message;
        if (type === 'sent') {
            p.classList.add('message-sent');
        } else if (type === 'received') {
            p.classList.add('message-received');
        } else if (type === 'error') {
            p.classList.add('error-message');
        } else { // 'status' par défaut
            p.classList.add('status-message');
        }
        messagesDisplay.appendChild(p);
        // Fait défiler automatiquement vers le bas
        messagesDisplay.scrollTop = messagesDisplay.scrollHeight;
    }

    // --- Gestion des événements WebSocket ---

    socket.onopen = function(event) {
        console.log('Connecté au serveur WebSocket.');
        connectionStatus.textContent = 'Statut : Connecté';
        connectionStatus.style.color = 'green';
        addMessageToDisplay('Connexion établie avec le serveur.', 'status');
    };

    socket.onmessage = function(event) {
        console.log('Message reçu du serveur:', event.data);
        addMessageToDisplay(`Reçu : "${event.data}"`, 'received');
    };

    socket.onerror = function(error) {
        console.error('Erreur WebSocket:', error);
        connectionStatus.textContent = 'Statut : Erreur de connexion';
        connectionStatus.style.color = 'red';
        addMessageToDisplay('Une erreur est survenue lors de la communication.', 'error');
    };

    socket.onclose = function(event) {
        console.log('Déconnecté du serveur WebSocket.');
        connectionStatus.textContent = 'Statut : Déconnecté';
        connectionStatus.style.color = 'orange';
        addMessageToDisplay('Connexion au serveur fermée.', 'status');
    };

    // --- Gestion de l'interface utilisateur ---

    sendButton.addEventListener('click', () => {
        const message = messageInput.value;
        if (message.trim() !== '') {
            if (socket.readyState === WebSocket.OPEN) {
                socket.send(message);
                addMessageToDisplay(`Envoyé : "${message}"`, 'sent'); // Affiche le message envoyé
                messageInput.value = '';
            } else {
                console.warn('La connexion WebSocket n\'est pas ouverte.');
                addMessageToDisplay('Impossible d\'envoyer : connexion non ouverte.', 'error');
            }
        }
    });

    messageInput.addEventListener('keypress', (event) => {
        if (event.key === 'Enter') {
            sendButton.click();
        }
    });
});
```



### Explications du Client

*   **`messagesDisplay` :** Un `div` est utilisé pour afficher une liste chronologique des messages envoyés et reçus, plutôt qu'une simple mise à jour du dernier message.
*   **`addMessageToDisplay(message, type)` :** Cette fonction utilitaire est introduite pour centraliser l'ajout de messages. Elle prend un `type` (ex: `'sent'`, `'received'`, `'status'`, `'error'`) pour appliquer des styles CSS différents et rendre la lecture plus claire.
*   **`socket.onopen`, `socket.onmessage`, `socket.onerror`, `socket.onclose` :** Ces gestionnaires utilisent `addMessageToDisplay` pour loguer les événements et les messages dans la zone d'affichage, avec des styles appropriés.
*   **`sendButton.addEventListener('click', ...)` :** Après avoir envoyé un message, `addMessageToDisplay` est appelé avec le type `'sent'` pour montrer à l'utilisateur ce qu'il vient d'envoyer.

Ces deux solutions offrent une base solide pour comprendre et implémenter les WebSockets. La deuxième solution, avec son suivi des messages, est souvent plus agréable pour l'utilisateur final et plus utile pour le débogage.