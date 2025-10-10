# Firebase Realtime Database et Firestore : présentation, modèles de données et sécurité

## 1. Présentation de Firebase

Firebase est une plateforme cloud proposée par Google, offrant des services managés spécialement conçus pour le développement d'applications web et mobiles avec des besoins temps réel et collaboration. Parmi ses services phares, on trouve **Realtime Database** et **Firestore**, deux bases de données NoSQL offrant des fonctionnalités temps réel mais avec des modèles et cas d'usage distincts.

---

## 2. Différences entre Firebase Realtime Database et Firestore

| Aspect                    | Firebase Realtime Database       | Firestore                        |
|--------------------------|---------------------------------|---------------------------------|
| **Type de base**          | Base de données JSON arborescente | Base documentaire NoSQL structurée en collections/documents |
| **Modèle de données**     | Arbre JSON unique                | Documents organisés en collections |
| **Requêtes**              | Limitées, basées sur chemins et filtres simples | Requêtes complexes avec filtres, ordres, agrégations |
| **Scalabilité**           | Scalabilité verticale            | Scalabilité horizontale automatique |
| **Isolation des données** | Moins fine                      | Forte isolation, plus modulaire |
| **Fonctionnalités temps réel** | Synchronisation en temps réel basée sur WebSocket | Synchronisation temps réel plus robuste avec écoutes filtrées |
| **Tarification**          | Basée sur données transférées et volumes stockés | Tarification plus fine par opération |

---

## 3. Modèles de données

### 3.1 Firebase Realtime Database

- Stockage sous forme d'un grand arbre JSON.
- Données dénormalisées : souvent besoin de dupliquer des données pour optimiser lectures.
- Exemple de structure JSON minimaliste :

```json
{
  "users": {
    "user1": {
      "name": "Alice",
      "age": 30,
      "messages": {
        "msg1": "Bonjour",
        "msg2": "Comment ça va?"
      }
    }
  }
}
```

### 3.2 Firestore

- Données organisées en **collections** et **documents**.
- Documents au format JSON mais stockés individuellement.
- Une collection contient plusieurs documents, chaque document peut contenir des sous-collections.
- Exemple :

```
users (collection)
 ├─ user1 (document)
 │    ├─ name: "Alice"
 │    ├─ age: 30
 │    └─ messages (subcollection)
 │          ├─ msg1 (document) : "Bonjour"
 │          └─ msg2 (document) : "Comment ça va?"
```

---

## 4. Sécurité et règles d'accès

Firebase intègre un système de règles de sécurité déclaratives garantissant que seul l’accès autorisé est possible.

### 4.1 Règles Realtime Database (extrait simple)

```json
{
  "rules": {
    "users": {
      "$user_id": {
        ".read": "$user_id === auth.uid",
        ".write": "$user_id === auth.uid"
      }
    }
  }
}
```
*Seul l’utilisateur authentifié peut lire et écrire ses données.*

### 4.2 Règles Firestore (extrait)

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

Ce langage permet une granularité fine et l’écriture de conditions complexes basées sur les métadonnées d’authentification, les données de la requête, ou l’heure.

---

## 5. Exemple simple d’utilisation Firestore en JavaScript

```javascript
import { initializeApp } from 'firebase/app';
import { getFirestore, collection, addDoc, onSnapshot } from 'firebase/firestore';

const firebaseConfig = {
  // Vos infos de config Firebase
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

async function addMessage(userId, text) {
  await addDoc(collection(db, 'users', userId, 'messages'), {
    text,
    timestamp: Date.now()
  });
}

function listenMessages(userId) {
  const messagesCol = collection(db, 'users', userId, 'messages');
  onSnapshot(messagesCol, (snapshot) => {
    snapshot.docChanges().forEach(change => {
      if(change.type === 'added') {
        console.log('Nouveau message:', change.doc.data());
      }
    });
  });
}
```

---

## 6. Synchronisation temps réel

Les deux bases propagent automatiquement les mises à jour aux clients abonnés via des connexions WebSocket.

- **Realtime Database** : envoie automatiquement les données modifiées sous forme d’arborescence.
- **Firestore** : permet d’écouter des documents ou collections, avec des notifications granuleuses (ajouts, modifications, suppressions).

---

## 7. Diagramme des interactions générales

```mermaid
graph TD
    Client1 -->|Read/Write| Firebase[Firestore | Realtime Database]
    Client2 -->|Read/Write| Firebase
    Firebase -->|Synchronisation temps réel| Client1
    Firebase -->|Synchronisation temps réel| Client2
```

---

## 8. Sources utilisées

- Documentation Firebase Realtime Database – [https://firebase.google.com/docs/database](https://firebase.google.com/docs/database)  
- Documentation Firestore – [https://firebase.google.com/docs/firestore](https://firebase.google.com/docs/firestore)  
- Guide règles de sécurité Firebase – [https://firebase.google.com/docs/rules](https://firebase.google.com/docs/rules)

---

Firebase offre une double approche de bases NoSQL en temps réel, chacune adaptée à des cas d'usage différents : Realtime Database pour des besoins simples à modèle arborescent, Firestore pour une gestion plus modulaire, scalable, avec des requêtes avancées. La gestion fine et intégrée des règles de sécurité garantit la protection et le contrôle des accès essentiels dans des applications collaboratives modernes.