Voici deux solutions complètes pour le TP sur la sécurisation d'une application collaborative en temps réel, en utilisant Node.js, Express, Socket.IO, JWT et Bcrypt.

---

# Solution 1 : Implémentation Directe des Consignes

Cette solution suit les étapes du TP de manière directe, en se concentrant sur la mise en œuvre des fonctionnalités d'authentification et d'autorisation.

## 1. Backend (Node.js/Express)

### Structure du Projet Backend



```
collaborative-notes/
├── node_modules/
├── package.json
├── package-lock.json
├── server.js
└── public/
    ├── index.html
    └── client.js
```



### `package.json` (dans `collaborative-notes/`)

Assurez-vous d'avoir ces dépendances installées :



```json
{
  "name": "collaborative-notes",
  "version": "1.0.0",
  "description": "Secured real-time collaborative notes app",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "bcrypt": "^5.1.1",
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.2",
    "socket.io": "^4.7.4"
  }
}
```



### `server.js` (dans `collaborative-notes/`)



```javascript
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const cors = require('cors'); // Pour permettre les requêtes du frontend

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
    cors: {
        origin: "*", // Permet toutes les origines pour le développement
        methods: ["GET", "POST", "PUT", "DELETE"]
    }
});

const PORT = process.env.PORT || 3000;
const JWT_SECRET = process.env.JWT_SECRET || 'super_secret_jwt_key'; // Clé secrète pour JWT
const BCRYPT_SALT_ROUNDS = 10; // Nombre de tours de salage pour bcrypt

// --- Données en mémoire (pour la simplicité du TP) ---
let users = []; // { id: 'uuid', username: 'user', password: 'hashed_password' }
let notes = []; // { id: 'uuid', content: 'text', authorId: 'uuid' }
let nextNoteId = 1;
let nextUserId = 1;

// --- Middlewares Express ---
app.use(cors()); // Active CORS pour toutes les routes
app.use(express.json()); // Pour parser les corps de requêtes JSON

// --- Middleware d'authentification JWT ---
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1]; // Format: Bearer TOKEN

    if (token == null) {
        return res.status(401).json({ message: 'Authentification requise.' });
    }

    jwt.verify(token, JWT_SECRET, (err, user) => {
        if (err) {
            console.error('Erreur de vérification JWT:', err.message);
            return res.status(403).json({ message: 'Token invalide ou expiré.' });
        }
        req.userId = user.id; // Attache l'ID de l'utilisateur à l'objet request
        next(); // Passe au middleware/route suivant
    });
};

// --- Routes d'authentification ---
app.post('/register', async (req, res) => {
    const { username, password } = req.body;
    if (!username || !password) {
        return res.status(400).json({ message: 'Pseudo et mot de passe sont requis.' });
    }
    if (users.find(u => u.username === username)) {
        return res.status(409).json({ message: 'Ce pseudo est déjà pris.' });
    }

    try {
        const hashedPassword = await bcrypt.hash(password, BCRYPT_SALT_ROUNDS);
        const newUser = { id: (nextUserId++).toString(), username, password: hashedPassword };
        users.push(newUser);
        console.log('Nouvel utilisateur enregistré:', newUser.username);
        res.status(201).json({ message: 'Utilisateur enregistré avec succès.' });
    } catch (error) {
        console.error('Erreur lors de l\'enregistrement:', error);
        res.status(500).json({ message: 'Erreur serveur lors de l\'enregistrement.' });
    }
});

app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    if (!username || !password) {
        return res.status(400).json({ message: 'Pseudo et mot de passe sont requis.' });
    }

    const user = users.find(u => u.username === username);
    if (!user) {
        return res.status(400).json({ message: 'Pseudo ou mot de passe incorrect.' });
    }

    try {
        const isMatch = await bcrypt.compare(password, user.password);
        if (!isMatch) {
            return res.status(400).json({ message: 'Pseudo ou mot de passe incorrect.' });
        }

        // Génère un JWT contenant l'ID de l'utilisateur
        const token = jwt.sign({ id: user.id, username: user.username }, JWT_SECRET, { expiresIn: '1h' });
        console.log('Utilisateur connecté:', user.username);
        res.status(200).json({ token, userId: user.id, username: user.username });
    } catch (error) {
        console.error('Erreur lors de la connexion:', error);
        res.status(500).json({ message: 'Erreur serveur lors de la connexion.' });
    }
});

// --- Routes API pour les notes ---

// GET toutes les notes (accessible à tous)
app.get('/notes', (req, res) => {
    res.status(200).json(notes);
});

// POST nouvelle note (authentifié seulement)
app.post('/notes', authenticateToken, (req, res) => {
    const { content } = req.body;
    if (!content) {
        return res.status(400).json({ message: 'Le contenu de la note est requis.' });
    }

    const newNote = {
        id: (nextNoteId++).toString(),
        content,
        authorId: req.userId, // L'ID de l'auteur est tiré du JWT
        authorUsername: users.find(u => u.id === req.userId)?.username || 'Inconnu'
    };
    notes.push(newNote);
    console.log('Note créée par', newNote.authorUsername, ':', newNote.content);
    io.emit('notes_updated', notes); // Diffuse la mise à jour en temps réel
    res.status(201).json(newNote);
});

// PUT mettre à jour une note (authentifié et propriétaire seulement)
app.put('/notes/:id', authenticateToken, (req, res) => {
    const { id } = req.params;
    const { content } = req.body;

    const noteIndex = notes.findIndex(n => n.id === id);
    if (noteIndex === -1) {
        return res.status(404).json({ message: 'Note non trouvée.' });
    }

    const note = notes[noteIndex];
    if (note.authorId !== req.userId) {
        return res.status(403).json({ message: 'Vous n\'êtes pas autorisé à modifier cette note.' });
    }

    if (!content) {
        return res.status(400).json({ message: 'Le contenu de la note est requis.' });
    }

    note.content = content;
    console.log('Note modifiée par', note.authorUsername, ':', note.content);
    io.emit('notes_updated', notes); // Diffuse la mise à jour en temps réel
    res.status(200).json(note);
});

// DELETE supprimer une note (authentifié et propriétaire seulement)
app.delete('/notes/:id', authenticateToken, (req, res) => {
    const { id } = req.params;

    const noteIndex = notes.findIndex(n => n.id === id);
    if (noteIndex === -1) {
        return res.status(404).json({ message: 'Note non trouvée.' });
    }

    const note = notes[noteIndex];
    if (note.authorId !== req.userId) {
        return res.status(403).json({ message: 'Vous n\'êtes pas autorisé à supprimer cette note.' });
    }

    notes.splice(noteIndex, 1);
    console.log('Note supprimée par', note.authorUsername, ':', note.id);
    io.emit('notes_updated', notes); // Diffuse la mise à jour en temps réel
    res.status(204).send(); // 204 No Content pour une suppression réussie
});

// --- Servir les fichiers statiques du frontend ---
app.use(express.static(path.join(__dirname, 'public')));

// Démarrage du serveur
server.listen(PORT, () => {
    console.log(`Serveur en écoute sur le port ${PORT}`);
});
```



### Explications du Serveur

*   **Dépendances :** `express` pour le serveur web, `socket.io` pour le temps réel, `bcrypt` pour le hachage des mots de passe, `jsonwebtoken` pour les JWT, et `cors` pour gérer les requêtes cross-origin du frontend.
*   **Données en mémoire :** `users` et `notes` sont des tableaux JavaScript. En production, cela serait une base de données.
*   **`JWT_SECRET` :** Une clé secrète est utilisée pour signer et vérifier les JWT. **En production, cette clé doit être forte et stockée en toute sécurité (ex: variable d'environnement).**
*   **`authenticateToken` Middleware :**
    *   Intercepte les requêtes.
    *   Extrait le JWT de l'en-tête `Authorization`.
    *   `jwt.verify()` : Vérifie la validité du token (signature, expiration).
    *   Si valide, attache l'`id` de l'utilisateur (issu du payload du JWT) à `req.userId`, le rendant disponible pour les routes suivantes.
    *   Renvoie 401 ou 403 si le token est manquant ou invalide.
*   **`/register` :** Hache le mot de passe avec `bcrypt.hash()` avant de stocker l'utilisateur.
*   **`/login` :** Compare le mot de passe fourni avec le hachage stocké via `bcrypt.compare()`. Si les identifiants sont corrects, un JWT est généré avec `jwt.sign()` et renvoyé au client.
*   **Routes `POST /notes`, `PUT /notes/:id`, `DELETE /notes/:id` :**
    *   Elles utilisent toutes le middleware `authenticateToken` pour s'assurer que l'utilisateur est connecté.
    *   `POST /notes` : Utilise `req.userId` pour définir l'auteur de la nouvelle note.
    *   `PUT /notes/:id` et `DELETE /notes/:id` : Vérifient que `note.authorId === req.userId` pour appliquer la règle de propriété des données.
*   **`io.emit('notes_updated', notes)` :** Après chaque modification des notes (ajout, modification, suppression), Socket.IO diffuse l'état actuel de toutes les notes à tous les clients connectés, assurant une mise à jour en temps réel.

## 2. Frontend (HTML/JavaScript)

### Structure du Projet Frontend



```
collaborative-notes/
└── public/
    ├── index.html
    └── client.js
```



### `public/index.html`



```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Notes Collaboratives Sécurisées</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background-color: #f4f4f4; color: #333; }
        .container { max-width: 900px; margin: 0 auto; background: white; padding: 25px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h1, h2 { text-align: center; color: #007bff; margin-bottom: 20px; }
        form { display: flex; flex-direction: column; gap: 10px; margin-bottom: 20px; padding: 15px; border: 1px solid #eee; border-radius: 5px; }
        input[type="text"], input[type="password"], textarea { padding: 10px; border: 1px solid #ddd; border-radius: 4px; font-size: 1em; }
        button { padding: 10px 15px; background-color: #28a745; color: white; border: none; border-radius: 4px; cursor: pointer; font-size: 1em; }
        button.delete { background-color: #dc3545; }
        button.edit { background-color: #ffc107; color: #333; }
        button:hover { opacity: 0.9; }
        .error-message { color: red; margin-top: 5px; }
        .success-message { color: green; margin-top: 5px; }
        #auth-section, #app-section { margin-bottom: 30px; }
        #app-section.hidden, #auth-section.hidden { display: none; }
        #notes-list { list-style-type: none; padding: 0; margin-top: 20px; }
        #notes-list li { background-color: #f9f9f9; border: 1px solid #eee; padding: 15px; margin-bottom: 10px; border-radius: 5px; display: flex; flex-direction: column; }
        #notes-list li .note-content { flex-grow: 1; margin-bottom: 10px; }
        #notes-list li .note-meta { font-size: 0.8em; color: #666; margin-bottom: 10px; }
        #notes-list li .note-actions { display: flex; gap: 5px; justify-content: flex-end; }
        .current-user-info { text-align: center; margin-bottom: 20px; font-weight: bold; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Tableau de Notes Collaboratif</h1>

        <div id="auth-section">
            <h2>Authentification</h2>
            <form id="register-form">
                <h3>S'inscrire</h3>
                <input type="text" id="register-username" placeholder="Pseudo" required>
                <input type="password" id="register-password" placeholder="Mot de passe" required>
                <button type="submit">S'inscrire</button>
                <p class="error-message" id="register-error"></p>
                <p class="success-message" id="register-success"></p>
            </form>
            <form id="login-form">
                <h3>Se connecter</h3>
                <input type="text" id="login-username" placeholder="Pseudo" required>
                <input type="password" id="login-password" placeholder="Mot de passe" required>
                <button type="submit">Se connecter</button>
                <p class="error-message" id="login-error"></p>
            </form>
        </div>

        <div id="app-section" class="hidden">
            <p class="current-user-info">Connecté en tant que : <span id="current-username-display"></span> (ID: <span id="current-userid-display"></span>)</p>
            <button id="logout-button">Se déconnecter</button>

            <h2>Ajouter une nouvelle note</h2>
            <form id="add-note-form">
                <textarea id="new-note-content" placeholder="Contenu de la note..." rows="3" required></textarea>
                <button type="submit">Ajouter la note</button>
                <p class="error-message" id="add-note-error"></p>
            </form>

            <h2>Notes existantes</h2>
            <ul id="notes-list">
                <!-- Les notes seront chargées ici par JavaScript -->
            </ul>
        </div>
    </div>

    <!-- Socket.IO client library -->
    <script src="/socket.io/socket.io.js"></script>
    <script src="client.js"></script>
</body>
</html>
```



### `public/client.js`



```javascript
const API_BASE_URL = 'http://localhost:3000'; // URL du backend
const socket = io(API_BASE_URL); // Connexion Socket.IO au serveur

// --- Références aux éléments DOM ---
const authSection = document.getElementById('auth-section');
const appSection = document.getElementById('app-section');
const registerForm = document.getElementById('register-form');
const loginForm = document.getElementById('login-form');
const addNoteForm = document.getElementById('add-note-form');
const notesList = document.getElementById('notes-list');
const logoutButton = document.getElementById('logout-button');
const currentUsernameDisplay = document.getElementById('current-username-display');
const currentUserIdDisplay = document.getElementById('current-userid-display');

// --- Variables d'état client ---
let currentUser = null; // { id: 'uuid', username: 'user' }
let currentToken = null;

// --- Fonctions utilitaires UI ---
function showSection(sectionId) {
    authSection.classList.add('hidden');
    appSection.classList.add('hidden');
    document.getElementById(sectionId).classList.remove('hidden');
}

function displayMessage(elementId, message, isError = false) {
    const element = document.getElementById(elementId);
    element.textContent = message;
    element.className = isError ? 'error-message' : 'success-message';
}

function clearMessages(elementId) {
    document.getElementById(elementId).textContent = '';
}

function getAuthHeaders() {
    return currentToken ? { 'Authorization': `Bearer ${currentToken}` } : {};
}

// --- Gestion de l'authentification côté client ---
function saveAuthData(token, userId, username) {
    localStorage.setItem('jwtToken', token);
    localStorage.setItem('userId', userId);
    localStorage.setItem('username', username);
    currentToken = token;
    currentUser = { id: userId, username: username };
    currentUsernameDisplay.textContent = username;
    currentUserIdDisplay.textContent = userId;
    showSection('app-section');
    fetchNotes(); // Recharger les notes après connexion
}

function clearAuthData() {
    localStorage.removeItem('jwtToken');
    localStorage.removeItem('userId');
    localStorage.removeItem('username');
    currentToken = null;
    currentUser = null;
    currentUsernameDisplay.textContent = '';
    currentUserIdDisplay.textContent = '';
    showSection('auth-section');
    fetchNotes(); // Recharger les notes pour les non-authentifiés
}

function loadAuthData() {
    const token = localStorage.getItem('jwtToken');
    const userId = localStorage.getItem('userId');
    const username = localStorage.getItem('username');
    if (token && userId && username) {
        currentToken = token;
        currentUser = { id: userId, username: username };
        currentUsernameDisplay.textContent = username;
        currentUserIdDisplay.textContent = userId;
        showSection('app-section');
    } else {
        showSection('auth-section');
    }
}

// --- Fonctions API ---
async function fetchNotes() {
    try {
        const response = await fetch(`${API_BASE_URL}/notes`, {
            headers: getAuthHeaders()
        });
        const data = await response.json();
        renderNotes(data);
    } catch (error) {
        console.error('Erreur lors du chargement des notes:', error);
        notesList.innerHTML = '<li>Erreur lors du chargement des notes.</li>';
    }
}

async function addNote(content) {
    clearMessages('add-note-error');
    try {
        const response = await fetch(`${API_BASE_URL}/notes`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                ...getAuthHeaders()
            },
            body: JSON.stringify({ content })
        });
        if (!response.ok) {
            const errorData = await response.json();
            throw new Error(errorData.message || 'Erreur lors de l\'ajout de la note.');
        }
        document.getElementById('new-note-content').value = '';
    } catch (error) {
        displayMessage('add-note-error', error.message, true);
    }
}

async function updateNote(id, content) {
    try {
        const response = await fetch(`${API_BASE_URL}/notes/${id}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
                ...getAuthHeaders()
            },
            body: JSON.stringify({ content })
        });
        if (!response.ok) {
            const errorData = await response.json();
            throw new Error(errorData.message || 'Erreur lors de la modification de la note.');
        }
    } catch (error) {
        alert('Erreur lors de la modification de la note: ' + error.message);
        console.error('Erreur lors de la modification de la note:', error);
    }
}

async function deleteNote(id) {
    if (!confirm('Êtes-vous sûr de vouloir supprimer cette note ?')) return;
    try {
        const response = await fetch(`${API_BASE_URL}/notes/${id}`, {
            method: 'DELETE',
            headers: getAuthHeaders()
        });
        if (!response.ok) {
            const errorData = await response.json();
            throw new Error(errorData.message || 'Erreur lors de la suppression de la note.');
        }
    } catch (error) {
        alert('Erreur lors de la suppression de la note: ' + error.message);
        console.error('Erreur lors de la suppression de la note:', error);
    }
}

// --- Rendu des notes ---
function renderNotes(notes) {
    notesList.innerHTML = '';
    notes.forEach(note => {
        const li = document.createElement('li');
        li.innerHTML = `
            <div class="note-content">${note.content}</div>
            <div class="note-meta">Auteur: ${note.authorUsername || 'Inconnu'}</div>
            <div class="note-actions"></div>
        `;

        const actionsDiv = li.querySelector('.note-actions');

        // Afficher les boutons Modifier/Supprimer uniquement si l'utilisateur est connecté ET est l'auteur
        if (currentUser && currentUser.id === note.authorId) {
            const editButton = document.createElement('button');
            editButton.textContent = 'Modifier';
            editButton.classList.add('edit');
            editButton.addEventListener('click', () => {
                const newContent = prompt('Modifier la note:', note.content);
                if (newContent !== null && newContent.trim() !== '') {
                    updateNote(note.id, newContent.trim());
                }
            });

            const deleteButton = document.createElement('button');
            deleteButton.textContent = 'Supprimer';
            deleteButton.classList.add('delete');
            deleteButton.addEventListener('click', () => deleteNote(note.id));

            actionsDiv.appendChild(editButton);
            actionsDiv.appendChild(deleteButton);
        }
        notesList.appendChild(li);
    });
}

// --- Écouteurs d'événements DOM ---
registerForm.addEventListener('submit', async (e) => {
    e.preventDefault();
    clearMessages('register-error');
    clearMessages('register-success');
    const username = document.getElementById('register-username').value;
    const password = document.getElementById('register-password').value;

    try {
        const response = await fetch(`${API_BASE_URL}/register`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ username, password })
        });
        const data = await response.json();
        if (response.ok) {
            displayMessage('register-success', data.message, false);
            registerForm.reset();
        } else {
            throw new Error(data.message || 'Erreur lors de l\'inscription.');
        }
    } catch (error) {
        displayMessage('register-error', error.message, true);
    }
});

loginForm.addEventListener('submit', async (e) => {
    e.preventDefault();
    clearMessages('login-error');
    const username = document.getElementById('login-username').value;
    const password = document.getElementById('login-password').value;

    try {
        const response = await fetch(`${API_BASE_URL}/login`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ username, password })
        });
        const data = await response.json();
        if (response.ok) {
            saveAuthData(data.token, data.userId, data.username);
            loginForm.reset();
        } else {
            throw new Error(data.message || 'Erreur lors de la connexion.');
        }
    } catch (error) {
        displayMessage('login-error', error.message, true);
    }
});

logoutButton.addEventListener('click', clearAuthData);

addNoteForm.addEventListener('submit', async (e) => {
    e.preventDefault();
    const content = document.getElementById('new-note-content').value.trim();
    if (content) {
        await addNote(content);
    }
});

// --- Écouteurs Socket.IO ---
socket.on('connect', () => {
    console.log('Connecté au serveur Socket.IO');
    // Si vous implémentez l'authentification Socket.IO, ce serait ici que vous enverriez le token
    // socket.emit('authenticate', { token: currentToken });
});

socket.on('disconnect', () => {
    console.log('Déconnecté du serveur Socket.IO');
});

socket.on('notes_updated', (updatedNotes) => {
    console.log('Notes mises à jour via Socket.IO');
    renderNotes(updatedNotes);
});

// --- Initialisation ---
document.addEventListener('DOMContentLoaded', () => {
    loadAuthData(); // Tente de charger les données d'authentification au démarrage
    fetchNotes(); // Charge les notes initiales
});
```



### Explications du Frontend

*   **`API_BASE_URL` et `socket` :** Configure l'URL du backend et la connexion Socket.IO.
*   **`currentUser` et `currentToken` :** Variables globales pour stocker l'état d'authentification de l'utilisateur.
*   **`localStorage` :** Le JWT, l'ID utilisateur et le pseudo sont stockés dans le `localStorage` du navigateur pour maintenir la session entre les rechargements de page.
*   **`getAuthHeaders()` :** Une fonction utilitaire qui retourne l'en-tête `Authorization` avec le JWT si l'utilisateur est connecté. Cet en-tête est inclus dans toutes les requêtes API nécessitant une authentification.
*   **`saveAuthData()` / `clearAuthData()` / `loadAuthData()` :** Gèrent la persistance et la récupération des informations d'authentification.
*   **`renderNotes(notes)` :**
    *   Parcourt le tableau des notes et crée les éléments HTML correspondants.
    *   **Contrôle d'accès UI :** Les boutons "Modifier" et "Supprimer" sont créés et ajoutés *uniquement* si `currentUser` est défini (utilisateur connecté) ET si `currentUser.id` correspond à `note.authorId`. Cela applique la règle de propriété des données côté client.
*   **Formulaires d'authentification :** Envoient les requêtes `POST /register` et `POST /login` au backend. En cas de succès de connexion, `saveAuthData` est appelée.
*   **`socket.on('notes_updated', ...)` :** Écoute l'événement de mise à jour des notes diffusé par le serveur et appelle `renderNotes()` pour rafraîchir l'affichage en temps réel.
*   **Initialisation :** Au chargement de la page, `loadAuthData()` tente de restaurer la session, puis `fetchNotes()` charge les notes.

---

# Solution 2 : Implémentation Structurée avec Authentification Socket.IO

Cette solution reprend la logique de la Solution 1 mais améliore la structure du code en séparant les préoccupations et en ajoutant une authentification basique pour les connexions Socket.IO.

## 1. Backend (Node.js/Express)

### Structure du Projet Backend



```
collaborative-notes-enhanced/
├── node_modules/
├── package.json
├── package-lock.json
├── server.js
├── authMiddleware.js
├── notesController.js
└── public/
    ├── index.html
    └── client.js
```



### `package.json` (dans `collaborative-notes-enhanced/`)

Identique à la Solution 1.

### `authMiddleware.js` (dans `collaborative-notes-enhanced/`)



```javascript
const jwt = require('jsonwebtoken');
const JWT_SECRET = process.env.JWT_SECRET || 'super_secret_jwt_key';

const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];

    if (token == null) {
        return res.status(401).json({ message: 'Authentification requise.' });
    }

    jwt.verify(token, JWT_SECRET, (err, user) => {
        if (err) {
            return res.status(403).json({ message: 'Token invalide ou expiré.' });
        }
        req.userId = user.id;
        req.username = user.username; // Attache aussi le username
        next();
    });
};

// Middleware pour l'authentification Socket.IO
const socketAuthMiddleware = (socket, next) => {
    const token = socket.handshake.auth.token;

    if (!token) {
        return next(new Error('Authentification Socket.IO requise.'));
    }

    jwt.verify(token, JWT_SECRET, (err, user) => {
        if (err) {
            return next(new Error('Token Socket.IO invalide ou expiré.'));
        }
        socket.userId = user.id; // Attache l'ID de l'utilisateur au socket
        socket.username = user.username; // Attache le username au socket
        next();
    });
};

module.exports = { authenticateToken, socketAuthMiddleware };
```



### `notesController.js` (dans `collaborative-notes-enhanced/`)



```javascript
// Données en mémoire (pour la simplicité du TP)
let notes = []; // { id: 'uuid', content: 'text', authorId: 'uuid', authorUsername: 'user' }
let nextNoteId = 1;

const getNotes = (req, res) => {
    res.status(200).json(notes);
};

const createNote = (req, res, io) => { // io est passé en paramètre pour la diffusion
    const { content } = req.body;
    if (!content) {
        return res.status(400).json({ message: 'Le contenu de la note est requis.' });
    }

    const newNote = {
        id: (nextNoteId++).toString(),
        content,
        authorId: req.userId,
        authorUsername: req.username // Utilise le username du JWT
    };
    notes.push(newNote);
    console.log('Note créée par', newNote.authorUsername, ':', newNote.content);
    io.emit('notes_updated', notes); // Diffuse la mise à jour
    res.status(201).json(newNote);
};

const updateNote = (req, res, io) => {
    const { id } = req.params;
    const { content } = req.body;

    const noteIndex = notes.findIndex(n => n.id === id);
    if (noteIndex === -1) {
        return res.status(404).json({ message: 'Note non trouvée.' });
    }

    const note = notes[noteIndex];
    if (note.authorId !== req.userId) {
        return res.status(403).json({ message: 'Vous n\'êtes pas autorisé à modifier cette note.' });
    }

    if (!content) {
        return res.status(400).json({ message: 'Le contenu de la note est requis.' });
    }

    note.content = content;
    console.log('Note modifiée par', note.authorUsername, ':', note.content);
    io.emit('notes_updated', notes);
    res.status(200).json(note);
};

const deleteNote = (req, res, io) => {
    const { id } = req.params;

    const noteIndex = notes.findIndex(n => n.id === id);
    if (noteIndex === -1) {
        return res.status(404).json({ message: 'Note non trouvée.' });
    }

    const note = notes[noteIndex];
    if (note.authorId !== req.userId) {
        return res.status(403).json({ message: 'Vous n\'êtes pas autorisé à supprimer cette note.' });
    }

    notes.splice(noteIndex, 1);
    console.log('Note supprimée par', note.authorUsername, ':', note.id);
    io.emit('notes_updated', notes);
    res.status(204).send();
};

module.exports = { getNotes, createNote, updateNote, deleteNote, notes }; // Exporte aussi 'notes' pour l'accès initial
```



### `server.js` (dans `collaborative-notes-enhanced/`)



```javascript
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const cors = require('cors');
const path = require('path');

const { authenticateToken, socketAuthMiddleware } = require('./authMiddleware');
const notesController = require('./notesController');

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
    cors: {
        origin: "*",
        methods: ["GET", "POST", "PUT", "DELETE"],
        credentials: true // Important si vous envoyez des cookies ou des en-têtes d'auth personnalisés
    },
    // Authentification Socket.IO: applique le middleware avant d'établir la connexion complète
    // io.use(socketAuthMiddleware); // Désactivé pour ce TP car le client n'envoie pas le token au handshake par défaut
});

const PORT = process.env.PORT || 3000;
const JWT_SECRET = process.env.JWT_SECRET || 'super_secret_jwt_key';
const BCRYPT_SALT_ROUNDS = 10;

// --- Données en mémoire pour les utilisateurs ---
let users = []; // { id: 'uuid', username: 'user', password: 'hashed_password' }
let nextUserId = 1;

// --- Middlewares Express ---
app.use(cors());
app.use(express.json());

// --- Routes d'authentification ---
app.post('/register', async (req, res) => {
    const { username, password } = req.body;
    if (!username || !password) {
        return res.status(400).json({ message: 'Pseudo et mot de passe sont requis.' });
    }
    if (users.find(u => u.username === username)) {
        return res.status(409).json({ message: 'Ce pseudo est déjà pris.' });
    }

    try {
        const hashedPassword = await bcrypt.hash(password, BCRYPT_SALT_ROUNDS);
        const newUser = { id: (nextUserId++).toString(), username, password: hashedPassword };
        users.push(newUser);
        console.log('Nouvel utilisateur enregistré:', newUser.username);
        res.status(201).json({ message: 'Utilisateur enregistré avec succès.' });
    } catch (error) {
        console.error('Erreur lors de l\'enregistrement:', error);
        res.status(500).json({ message: 'Erreur serveur lors de l\'enregistrement.' });
    }
});

app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    if (!username || !password) {
        return res.status(400).json({ message: 'Pseudo et mot de passe sont requis.' });
    }

    const user = users.find(u => u.username === username);
    if (!user) {
        return res.status(400).json({ message: 'Pseudo ou mot de passe incorrect.' });
    }

    try {
        const isMatch = await bcrypt.compare(password, user.password);
        if (!isMatch) {
            return res.status(400).json({ message: 'Pseudo ou mot de passe incorrect.' });
        }

        const token = jwt.sign({ id: user.id, username: user.username }, JWT_SECRET, { expiresIn: '1h' });
        console.log('Utilisateur connecté:', user.username);
        res.status(200).json({ token, userId: user.id, username: user.username });
    } catch (error) {
        console.error('Erreur lors de la connexion:', error);
        res.status(500).json({ message: 'Erreur serveur lors de la connexion.' });
    }
});

// --- Routes API pour les notes (utilisant le contrôleur) ---
app.get('/notes', notesController.getNotes);
app.post('/notes', authenticateToken, (req, res) => notesController.createNote(req, res, io));
app.put('/notes/:id', authenticateToken, (req, res) => notesController.updateNote(req, res, io));
app.delete('/notes/:id', authenticateToken, (req, res) => notesController.deleteNote(req, res, io));

// --- Servir les fichiers statiques du frontend ---
app.use(express.static(path.join(__dirname, 'public')));

// --- Logique Socket.IO ---
io.on('connection', (socket) => {
    console.log(`Socket connecté: ${socket.id}`);
    // Si socketAuthMiddleware est activé, socket.userId et socket.username seraient disponibles ici
    // console.log(`Utilisateur Socket authentifié: ${socket.username} (ID: ${socket.userId})`);

    socket.on('disconnect', () => {
        console.log(`Socket déconnecté: ${socket.id}`);
    });

    // Exemple d'événement Socket.IO qui pourrait nécessiter une authentification
    // socket.on('create_note_ws', (data) => {
    //     if (!socket.userId) { // Vérifie si le socket est authentifié
    //         return socket.emit('error', { message: 'Non authentifié pour créer une note via WS.' });
    //     }
    //     // Logique de création de note ici, similaire à la route POST /notes
    // });
});

// Démarrage du serveur
server.listen(PORT, () => {
    console.log(`Serveur en écoute sur le port ${PORT}`);
});
```



### Explications du Backend

*   **Séparation des préoccupations :**
    *   `authMiddleware.js` : Contient le middleware `authenticateToken` pour Express et un `socketAuthMiddleware` pour Socket.IO.
    *   `notesController.js` : Contient les fonctions de gestion CRUD pour les notes.
*   **`socketAuthMiddleware` :** Ce middleware est conçu pour être utilisé avec `io.use(socketAuthMiddleware)` pour authentifier les connexions Socket.IO dès le handshake. Il vérifie un token envoyé dans `socket.handshake.auth.token`. Si le token est valide, `socket.userId` et `socket.username` sont attachés à l'objet `socket`.
    *   **Note :** Pour ce TP, il est commenté dans `server.js` car le client par défaut n'envoie pas le token au handshake. Pour l'activer, le client devrait faire `io({ auth: { token: currentToken } })`.
*   **Contrôleurs de notes :** Les fonctions du `notesController` prennent `req`, `res` et `io` en paramètres, permettant d'accéder aux données de la requête, d'envoyer la réponse et de diffuser les mises à jour.
*   Le reste de la logique est similaire à la Solution 1, mais mieux organisée.

## 2. Frontend (HTML/JavaScript)

Le frontend est très similaire à la Solution 1, avec une légère modification pour potentiellement envoyer le token lors de la connexion Socket.IO si le `socketAuthMiddleware` était activé côté serveur.

### `public/index.html`

Identique à la Solution 1.

### `public/client.js`



```javascript
const API_BASE_URL = 'http://localhost:3000';

// --- Connexion Socket.IO avec authentification (si activée côté serveur) ---
// Si socketAuthMiddleware est activé sur le serveur, décommenter la ligne suivante
// const socket = io(API_BASE_URL, { auth: { token: localStorage.getItem('jwtToken') } });
const socket = io(API_BASE_URL); // Par défaut sans auth au handshake

// --- Références aux éléments DOM (identiques à Solution 1) ---
const authSection = document.getElementById('auth-section');
const appSection = document.getElementById('app-section');
const registerForm = document.getElementById('register-form');
const loginForm = document.getElementById('login-form');
const addNoteForm = document.getElementById('add-note-form');
const notesList = document.getElementById('notes-list');
const logoutButton = document.getElementById('logout-button');
const currentUsernameDisplay = document.getElementById('current-username-display');
const currentUserIdDisplay = document.getElementById('current-userid-display');

// --- Variables d'état client (identiques à Solution 1) ---
let currentUser = null;
let currentToken = null;

// --- Fonctions utilitaires UI (identiques à Solution 1) ---
function showSection(sectionId) { /* ... */ }
function displayMessage(elementId, message, isError = false) { /* ... */ }
function clearMessages(elementId) { /* ... */ }
function getAuthHeaders() { /* ... */ }

// --- Gestion de l'authentification côté client (identiques à Solution 1) ---
function saveAuthData(token, userId, username) {
    localStorage.setItem('jwtToken', token);
    localStorage.setItem('userId', userId);
    localStorage.setItem('username', username);
    currentToken = token;
    currentUser = { id: userId, username: username };
    currentUsernameDisplay.textContent = username;
    currentUserIdDisplay.textContent = userId;
    showSection('app-section');
    fetchNotes();
    // Si socketAuthMiddleware est activé, on pourrait reconnecter le socket ici avec le nouveau token
    // socket.auth.token = token;
    // socket.connect();
}

function clearAuthData() {
    localStorage.removeItem('jwtToken');
    localStorage.removeItem('userId');
    localStorage.removeItem('username');
    currentToken = null;
    currentUser = null;
    currentUsernameDisplay.textContent = '';
    currentUserIdDisplay.textContent = '';
    showSection('auth-section');
    fetchNotes();
    // Si socketAuthMiddleware est activé, on pourrait déconnecter le socket ici
    // socket.disconnect();
}

function loadAuthData() {
    const token = localStorage.getItem('jwtToken');
    const userId = localStorage.getItem('userId');
    const username = localStorage.getItem('username');
    if (token && userId && username) {
        currentToken = token;
        currentUser = { id: userId, username: username };
        currentUsernameDisplay.textContent = username;
        currentUserIdDisplay.textContent = userId;
        showSection('app-section');
    } else {
        showSection('auth-section');
    }
}

// --- Fonctions API (identiques à Solution 1) ---
async function fetchNotes() { /* ... */ }
async function addNote(content) { /* ... */ }
async function updateNote(id, content) { /* ... */ }
async function deleteNote(id) { /* ... */ }

// --- Rendu des notes (identiques à Solution 1) ---
function renderNotes(notes) { /* ... */ }

// --- Écouteurs d'événements DOM (identiques à Solution 1) ---
registerForm.addEventListener('submit', async (e) => { /* ... */ });
loginForm.addEventListener('submit', async (e) => { /* ... */ });
logoutButton.addEventListener('click', clearAuthData);
addNoteForm.addEventListener('submit', async (e) => { /* ... */ });

// --- Écouteurs Socket.IO (identiques à Solution 1) ---
socket.on('connect', () => {
    console.log('Connecté au serveur Socket.IO');
    // Si socketAuthMiddleware est activé sur le serveur, le token est envoyé au handshake
    // Si ce n'est pas le cas, vous pourriez envoyer un événement d'authentification ici après la connexion
    // if (currentToken) {
    //     socket.emit('authenticate', { token: currentToken });
    // }
});

socket.on('disconnect', () => {
    console.log('Déconnecté du serveur Socket.IO');
});

socket.on('notes_updated', (updatedNotes) => {
    console.log('Notes mises à jour via Socket.IO');
    renderNotes(updatedNotes);
});

// --- Initialisation (identiques à Solution 1) ---
document.addEventListener('DOMContentLoaded', () => {
    loadAuthData();
    fetchNotes();
});
```



### Explications du Frontend

*   Le code frontend est très similaire à la Solution 1. La principale différence réside dans la possibilité d'envoyer le JWT lors de la connexion Socket.IO si le `socketAuthMiddleware` est activé côté serveur.
*   Les commentaires dans `client.js` indiquent où les modifications seraient nécessaires pour activer l'authentification Socket.IO au handshake ou via un événement `authenticate`.

---

## Instructions de Lancement et Test (pour les deux solutions)

1.  **Installation des dépendances :**
    *   Naviguez dans le dossier `collaborative-notes/` (ou `collaborative-notes-enhanced/`).
    *   Exécutez `npm install`.

2.  **Lancement du serveur :**
    *   Dans le terminal, exécutez `node server.js`.
    *   Le serveur devrait démarrer et écouter sur `http://localhost:3000`.

3.  **Lancement du client :**
    *   Ouvrez votre navigateur web et accédez au fichier `public/index.html` (par exemple, en le glissant-déposant dans le navigateur, ou en utilisant un serveur HTTP simple comme `http-server public -p 8080` puis en allant sur `http://localhost:8080`).

4.  **Scénarios de test :**
    *   **Non authentifié :** Tentez d'ajouter, modifier ou supprimer une note. Vous devriez recevoir des erreurs 401/403 dans la console du navigateur et les actions ne devraient pas être possibles. Vous devriez pouvoir voir les notes existantes.
    *   **Inscription :** Créez un nouvel utilisateur (ex: `userA`/`passA`).
    *   **Connexion :** Connectez-vous avec `userA`. Les formulaires d'authentification devraient disparaître, et les boutons "Modifier" et "Supprimer" devraient apparaître pour les notes que `userA` crée.
    *   **Création de note :** Ajoutez une note avec `userA`.
    *   **Modification/Suppression par l'auteur :** Modifiez et supprimez la note créée par `userA`. Cela devrait fonctionner.
    *   **Test de propriété (avec deux utilisateurs) :**
        *   Ouvrez un deuxième onglet de navigateur (ou un navigateur différent) et connectez-vous avec un nouvel utilisateur `userB`/`passB`.
        *   `userB` devrait voir la note de `userA`.
        *   Tentez de modifier ou supprimer la note de `userA` avec `userB`. Cela devrait échouer avec une erreur 403.
        *   Créez une note avec `userB`. `userA` devrait la voir en temps réel.
        *   `userA` ne devrait pas pouvoir modifier/supprimer la note de `userB`.
    *   **Déconnexion :** Déconnectez-vous. Les boutons d'action devraient disparaître, et les formulaires d'authentification réapparaître.

Ces solutions fournissent une base solide pour comprendre et mettre en œuvre la sécurité dans les applications collaboratives en temps réel. La Solution 2, avec sa structure modulaire et son exemple d'authentification Socket.IO, est plus proche des pratiques de développement professionnel.