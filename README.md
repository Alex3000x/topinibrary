# Topinibrary 📚

Applicazione web full-stack per la gestione e consultazione del catalogo di una biblioteca digitale, con autenticazione JWT, controllo degli accessi basato sui ruoli (RBAC) e pannello amministrativo per operazioni CRUD su libri e utenti.

Progetto sviluppato in team da 2 persone come Project Work per il corso **Full Stack Software Developer** presso **ITS Adriano Olivetti (Modena)**.

---

## 🎯 Contesto e Contributo Personale

Il progetto è stato realizzato in collaborazione tra due sviluppatori con una divisione chiara delle responsabilità:

- **Architettura Backend & API REST (Focus principale):** progettazione e sviluppo dell'intero server Node.js/Express (`server/`), modellazione delle collection MongoDB (`books`, `users`), implementazione del flusso di autenticazione con **JWT (Access + Refresh Token)**, hashing delle password con `bcrypt`, middleware di autorizzazione per utenti standard e amministratori, e logica server-side per la verifica delle copertine tramite API Open Library.
- **Integrazione Full-Stack & Frontend Support:** supporto allo sviluppo e al collegamento dei componenti React (`onlinelibrary/`) con gli endpoint REST, gestione dello stato di autenticazione lato client e rifinitura dei flussi CRUD e sicurezza account.

---

## 🛠️ Stack Tecnologico

| Livello | Tecnologie |
| :--- | :--- |
| **Frontend** | React 19, Vite, React Bootstrap, Bootstrap 5, Axios, Swiper |
| **Backend** | Node.js (ES Modules), Express 5 |
| **Database** | MongoDB (driver nativo `mongodb`) |
| **Sicurezza & Auth** | JSON Web Tokens (`jsonwebtoken`), `bcrypt`, CORS |
| **Dev Tooling** | `concurrently`, `nodemon`, ESLint |

---

## ✨ Funzionalità Principali

### Catalogo & Ricerca (Pubblico)
- **Home Page dinamica:** caroselli interattivi (*Scelti dallo staff* e *Nuovi arrivi*) alimentati dal catalogo.
- **Ricerca multi-criterio con debounce:** filtro in tempo reale (400ms) su tutti i campi o per singolo attributo (Titolo, Autore, Anno, Genere, ISBN, Editore, Lingua).
- **Ordinamento e paginazione:** ordinamento client-side (novità, titolo, autore, anno di pubblicazione) e paginazione a 12 elementi per vista.
- **Validazione intelligente delle copertine:** durante l'inserimento o la modifica di un libro, il backend interroga Open Library tramite ISBN e verifica il peso in byte del buffer restituito, sostituendo automaticamente i placeholder vuoti (1x1 px) o gli ISBN assenti con un'immagine di fallback locale.

### Autenticazione & Gestione Account (Utenti Registrati)
- Registrazione e login con rilascio di **Access Token** (15m) e **Refresh Token** (7d).
- Modifica dei dati anagrafici e dell'avatar utente.
- Cambio email (con verifica della password attuale), cambio password e cancellazione definitiva dell'account.

### Area Riservata (Admin)
- **Gestione Catalogo:** aggiunta, modifica ed eliminazione dei libri con calcolo automatico dello stato di disponibilità in base al numero di copie.
- **Gestione Utenti:** visualizzazione degli utenti registrati e promozione/revoca dei privilegi amministrativi (con blocco di sicurezza per impedire all'admin di revocare i propri permessi).

---

## 📁 Struttura del Repository

```text
topinibrary/
├── package.json              # Script root (avvio simultaneo client + server con concurrently)
├── onlinelibrary/            # Client SPA React + Vite
│   └── src/
│       ├── components/       # Componenti UI, Modali CRUD, Auth, Profilo e Admin
│       ├── App.jsx           # Routing di vista, stato globale utente e chiamate CRUD
│       └── main.jsx
└── server/                   # Backend REST API Express
    ├── db.js                 # Connessione singleton a MongoDB
    └── server.js             # Endpoint REST, middleware JWT e logica business
```

---

## 🚀 Installazione e Avvio in Locale

### Prerequisiti
- **Node.js** (v18+ consigliata)
- **MongoDB** in esecuzione in locale sulla porta di default (`mongodb://127.0.0.1:27017`)

### Setup

1. **Clona il repository:**
   ```bash
   git clone https://github.com/Alex3000x/topinibrary.git
   cd topinibrary
   ```

2. **Installa le dipendenze (root, server e frontend):**
   ```bash
   npm install
   npm install --prefix server
   npm install --prefix onlinelibrary
   ```

3. **Avvia l'applicazione in modalità sviluppo:**
   Dalla cartella principale esegui:
   ```bash
   npm run dev
   ```
   Il comando avvia contemporaneamente:
   - **Backend API:** `http://localhost:3000`
   - **Frontend Vite:** `http://localhost:5173`

> **Nota per testare le funzionalità Admin:** alla registrazione ogni nuovo utente viene creato con `isAdmin: false`. Per accedere alle funzionalità amministrative in locale, imposta `isAdmin: true` sul documento utente nella collection `users` del database `librarydb` su MongoDB.

---

## 🔌 Principali Endpoint API

Base URL: `http://localhost:3000/topinibrary`

| Metodo | Endpoint | Accesso | Descrizione |
| :--- | :--- | :--- | :--- |
| `GET` | `/books` | Pubblico | Recupera il catalogo con filtri query (`all`, `title`, `author`, `genre`, ecc.) |
| `POST` | `/books` | Admin (JWT) | Aggiunge un nuovo libro con validazione copertina Open Library |
| `PUT` | `/books/:id` | Admin (JWT) | Aggiorna i dettagli di un libro esistente |
| `DELETE` | `/books/:id` | Admin (JWT) | Rimuove un libro dal catalogo |
| `POST` | `/register` | Pubblico | Registra un nuovo utente con password hashata (`bcrypt`) |
| `POST` | `/login` | Pubblico | Autentica l'utente e restituisce `accessToken` e `refreshToken` |
| `POST` | `/refresh` | Pubblico | Rinnova l'`accessToken` tramite `refreshToken` |
| `PUT` | `/users/:id` | Utente / Admin | Aggiorna i dati del profilo utente |
| `PATCH` | `/users/:id/email` | Proprietario | Aggiorna l'email previa verifica password |
| `PATCH` | `/users/:id/password` | Proprietario | Aggiorna la password previa verifica della vecchia |
| `DELETE` | `/users/:id` | Proprietario / Admin | Elimina l'account utente |
| `GET` | `/users` | Admin (JWT) | Elenca tutti gli utenti (escludendo l'hash della password) |
| `PATCH` | `/admin/users/:id/role` | Admin (JWT) | Modifica il flag `isAdmin` di un utente |

---

## 🔮 Miglioramenti Futuri (Roadmap Tecnica)

- Spostamento delle chiavi segrete JWT e della URI MongoDB in variabili d'ambiente (`.env`).
- Implementazione di un interceptor Axios lato client per il rinnovo automatico dell'`accessToken` alla scadenza tramite `/refresh`.
- Aggiunta della sezione *Preferiti* per gli utenti autenticati e gestione dei prestiti.
- Introduzione di test di integrazione sugli endpoint REST.
