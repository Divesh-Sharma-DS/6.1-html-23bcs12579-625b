# 6.1-html-23bcs12579-625b
 Demonstrate Middleware implementation (logging/auth).

<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Middleware Demo — Logging & Auth</title>
<style>
body { font-family: system-ui, -apple-system, 'Segoe UI', Roboto, Arial; padding: 24px; }
button { margin: 6px 0; }
pre { background:#f3f3f3; padding:12px; border-radius:6px; }
.row { display:flex; gap:12px; align-items:center; }
input { padding:6px; }
</style>
</head>
<body>
<h1>Middleware demo — logging & auth</h1>


<section>
<h2>Login (demo)</h2>
<div class="row">
<input id="username" placeholder="username (admin)" value="admin" />
<input id="password" type="password" placeholder="password (password)" value="password" />
<button id="loginBtn">Login</button>
<button id="logoutBtn">Logout</button>
</div>
<p>Token: <code id="tokenDisplay">(not logged in)</code></p>
</section>


<section>
<h2>API calls</h2>
<button id="publicBtn">GET /api/public</button>
<button id="protectedBtn">GET /api/protected</button>
<button id="updateBtn">POST /api/protected-update</button>
<pre id="output"></pre>
</section>


<script>
const out = document.getElementById('output');
const tokenDisplay = document.getElementById('tokenDisplay');


function log(msg) {
const time = new Date().toLocaleTimeString();
out.textContent = `[${time}] ${msg}\n` + out.textContent;
}


function saveToken(t) {
if (t) {
localStorage.setItem('demo_token', t);
tokenDisplay.textContent = t;
} else {
localStorage.removeItem('demo_token');
tokenDisplay.textContent = '(not logged in)';
}
}


// initialize
const existing = localStorage.getItem('demo_token');
if (existing) tokenDisplay.textContent = existing;


document.getElementById('loginBtn').addEventListener('click', async () => {
const username = document.getElementById('username').value;
const password = document.getElementById('password').value;
try {
const r = await fetch('/api/login', {
method: 'POST',
</html>



server.js
 // server.js
const express = require('express');
const path = require('path');
const bodyParser = require('body-parser');
const cors = require('cors');


const app = express();
const PORT = process.env.PORT || 3000;


app.use(cors());
app.use(bodyParser.json());


// --- Logging middleware (custom) ---
// Logs method, url, timestamp and optionally request body for POST/PUT
function loggingMiddleware(req, res, next) {
const now = new Date().toISOString();
const { method, originalUrl } = req;
const bodyPreview = (method === 'POST' || method === 'PUT') ? ` body=${JSON.stringify(req.body)}` : '';
console.log(`[${now}] ${method} ${originalUrl}${bodyPreview}`);
next();
}
app.use(loggingMiddleware);


// --- Simple authentication middleware ---
// For demo purposes we expect header: Authorization: Bearer demo-token
// In a real app replace with JWT or sessions and secure storage
function authMiddleware(req, res, next) {
const authHeader = req.headers['authorization'] || '';
if (!authHeader.startsWith('Bearer ')) {
return res.status(401).json({ error: 'Missing or invalid Authorization header' });
}
const token = authHeader.slice(7).trim();
// Very simple token check
if (token !== 'demo-token') {
return res.status(403).json({ error: 'Invalid token' });
}
// attach user info to request for downstream handlers
req.user = { id: 'demo-user', name: 'Demo User' };
next();
}


// --- Routes ---
// Serve the static front-end (index.html) from /public
app.use(express.static(path.join(__dirname, 'public')));


// Public API route (no auth)
app.get('/api/public', (req, res) => {
res.json({ message: 'This is public data. No auth required.' });
});


// Protected API route (requires authMiddleware)
app.get('/api/protected', authMiddleware, (req, res) => {
res.json({ message: `Hello ${req.user.name}. This is protected data.` });
});


// Login route (demo only)
// Accepts { username, password } and returns token if "correct"
app.post('/api/login', (req, res) => {
const { username, password } = req.body || {};
// Demo rule: username === 'admin' && password === 'password' -> returns demo-token
});

 
 
 
 file pacakage.json

{
"name": "middleware-demo",
"version": "1.0.0",
"main": "server.js",
"scripts": {
"start": "node server.js"
},
"dependencies": {
"body-parser": "^1.20.0",
"cors": "^2.8.5",
"express": "^4.18.2"
}
}
