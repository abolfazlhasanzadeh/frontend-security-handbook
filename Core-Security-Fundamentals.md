
# Chapter 1: Core Security Fundamentals

## 1-1: Understanding the Origin Concept

### What is Origin?

The **Origin** is a fundamental security concept in web browsers. It consists of three parts:

```
https://example.com:443
└┬┘ └──┬──┘   └┬┘
Protocol   Host          Port
```

Origin = Protocol + Host + Port

### Key Points:
- Two URLs with same protocol, host, and port = **Same Origin**
- Example: `https://example.com` and `https://example.com:443` → Same Origin
- Example: `https://example.com` and `http://example.com` → **Different Origin** (different protocol)

### Why It Matters (Frontend):
- Browsers use Origin to enforce **Same-Origin Policy (SOP)**
- SOP restricts how frontend code from one origin can interact with resources from another origin
- This is the foundation of browser security model that every frontend developer must understand

---

## 1-2: Deep Dive into Origin

### Same-Origin Policy (SOP) in Depth

SOP allows:
- Frontend scripts from same origin to access each other's DOM and data
- Cross-origin writes (like links, redirects, form submissions)
- Cross-origin embedding (images, scripts, stylesheets, iframes)

SOP blocks:
- Cross-origin reads (frontend cannot read response from cross-origin requests)
- Cross-origin DOM access

### Exceptions to SOP (Frontend-Controlled):
- **CORS (Cross-Origin Resource Sharing)** — Server allows cross-origin requests
- **postMessage API** — Secure cross-origin communication
- **JSONP** — Legacy workaround (now considered insecure)

### CORS in Frontend:
- Browser automatically sends `Origin` header in requests
- Browser checks server response header `Access-Control-Allow-Origin`
- If mismatch, browser blocks frontend from accessing the response

---

## 1-3: Storage Mechanisms in Origin

### Browser Storage Types (Frontend Accessible)

| Storage Type | Scope | Persistence | Size Limit | JS Access |
|--------------|-------|-------------|------------|-----------|
| **localStorage** | Per origin | Until manually cleared | ~5-10MB |  Full |
| **sessionStorage** | Per tab | Until tab closes | ~5-10MB |  Full |
| **Cookies** | Per origin | Set by expiration | ~4KB |  Limited |
| **IndexedDB** | Per origin | Until manually cleared | Large |  Full |

### Security Considerations for Frontend:

**localStorage & sessionStorage:**
- Accessible to any JavaScript from the same origin
- **Vulnerable to XSS attacks** — any injected script can read them
- Never store tokens, passwords, or sensitive data here

**Cookies:**
- Can be marked `HttpOnly` → **inaccessible to JavaScript** (safe from XSS)
- Can be marked `Secure` → only sent over HTTPS
- Can be marked `SameSite` → controls cross-site sending (CSRF protection)

### Frontend Best Practice:
```javascript
// ❌ DON'T: Store tokens in localStorage
localStorage.setItem('token', 'eyJhbGciOiJIUzI1NiIs...');

// ✅ DO: Use HttpOnly cookies (set by backend)
// Or store in memory (React state, Vuex, Redux)
```

---

## 1-4: What is Broadcast Channel?

### Overview

**BroadcastChannel API** enables communication between different tabs/windows of the same origin — all from frontend code.

```javascript
// Tab 1
const channel = new BroadcastChannel('my-channel');
channel.postMessage('Hello from Tab 1!');

// Tab 2
const channel = new BroadcastChannel('my-channel');
channel.onmessage = (event) => {
  console.log(event.data); // 'Hello from Tab 1!'
};
```

### Security Implications for Frontend:
- Only works within same origin
- Messages are not encrypted
- Sensitive data should not be sent through broadcast channels

### Frontend Use Cases:
- Single sign-out across tabs
- Real-time updates syncing
- Cache invalidation across tabs
- Sharing authentication state between tabs

---

## 1-5: Cookie Management in Origin

### Cookie Attributes (Frontend Must Know)

| Attribute | Purpose | Frontend Impact |
|-----------|---------|-----------------|
| **HttpOnly** | Prevents JavaScript access | ✅ Protects from XSS token theft |
| **Secure** | Only sent over HTTPS | ✅ Prevents MITM attacks |
| **SameSite** | Controls cross-origin sending | ✅ Prevents CSRF attacks |
| **Domain** | Specifies allowed domains | Controls cookie scope |
| **Path** | Limits URL path scope | Controls cookie scope |
| **Max-Age** | Sets expiration time | Controls cookie lifetime |

### Frontend Cookie Handling:

```javascript
// Reading cookies (if not HttpOnly)
document.cookie // Returns all non-HttpOnly cookies

// Setting cookies (not recommended for sensitive data)
document.cookie = 'theme=dark; Secure; SameSite=Lax; Max-Age=86400';

// Sending cookies automatically with fetch
fetch('/api/data', {
  credentials: 'include' // Include cookies in request
});
```

### Frontend Best Practices:
1. Never access authentication cookies via JavaScript
2. Use `credentials: 'include'` for authenticated requests
3. Don't store sensitive data in client-side cookies
4. Clear cookies on logout

---

## 1-6: What is Hash?

### Definition

A **hash function** is a mathematical algorithm that converts input of any size into a fixed-size output (hash value).

```
Input (any size) → Hash Function → Output (fixed size)
"password123"  →     SHA-256     → "ef92b778bafbcf4d7a..."
```

### Key Properties:
- **Deterministic**: Same input → Same output
- **Fixed size**: Output length is constant
- **One-way**: Cannot reverse from output to input
- **Collision-resistant**: Different inputs rarely produce same output
- **Fast to compute**

### Where Frontend Uses Hash:
- Password hashing (should be done on backend, but frontend should understand)
- Integrity verification of scripts (SRI — Subresource Integrity)
- Checksums for file uploads
- Cache busting

### Common Hash Algorithms (Frontend Context):
| Algorithm | Output Size | Use in Frontend |
|-----------|-------------|-----------------|
| MD5 | 128-bit | ❌ Broken, avoid |
| SHA-1 | 160-bit | ❌ Deprecated |
| SHA-256 | 256-bit | ✅ SRI, file integrity |
| SHA-512 | 512-bit | ✅ SRI, file integrity |

---

## 1-7: Hash Lookup Concept

### What is Hash Lookup?

Hash lookup is using a hash as a key to efficiently find stored values.

### Frontend Use Cases:

**1. Cache Management:**
```javascript
const cache = new Map();

async function getData(url) {
  const hash = await sha256(url);
  if (cache.has(hash)) {
    return cache.get(hash); // O(1) lookup
  }
  const data = await fetch(url).then(r => r.json());
  cache.set(hash, data);
  return data;
}
```

**2. Deduplication:**
```javascript
function calculateFileHash(file) {
  // Use crypto.subtle to hash file
  // Store hash to avoid duplicate uploads
}
```

### Frontend Security Consideration:
- Hash lookups in frontend don't expose sensitive data
- Used for performance, not security

---

## 1-8: Hash vs. Encoding

### Key Differences

| Aspect | Hash | Encoding |
|--------|------|----------|
| **Purpose** | Integrity, fingerprinting | Data representation |
| **Reversibility** | ❌ One-way (irreversible) | ✅ Two-way (reversible) |
| **Key Required** | ❌ No key needed | ❌ No key needed |
| **Output Size** | Fixed | Variable |
| **Collisions** | Possible (rare) | None |
| **Frontend Use** | SRI, file checksums | Base64, URL encoding |

### Frontend Examples:

**Encoding:**
```javascript
// Base64 encoding (reversible)
const encoded = btoa('Hello World'); // "SGVsbG8gV29ybGQ="
const decoded = atob(encoded); // "Hello World"

// URL encoding
const url = encodeURIComponent('name=John Doe');
```

**Hashing:**
```javascript
// Hashing (irreversible)
async function hashMessage(msg) {
  const encoder = new TextEncoder();
  const data = encoder.encode(msg);
  const hash = await crypto.subtle.digest('SHA-256', data);
  return Array.from(new Uint8Array(hash)).map(b => b.toString(16).padStart(2, '0')).join('');
}
```

### Key Takeaway:
- Encoding is for **transmission** (reversible)
- Hashing is for **integrity** (irreversible)
- Neither provides encryption/confidentiality

---

## 1-9: Password Storage with Hashing (Frontend Context)

### Frontend Role in Password Security

**Important:** Password hashing should happen on the **backend**, but frontend developers must understand it to implement proper authentication flows.

### The Password Flow:

```
User → Enters Password → Browser → HTTPS → Server → Hash + Salt → Database
```

### Frontend Responsibilities:

1. **Transmit password securely** via HTTPS
2. **Never hash on client-side** — it defeats the purpose
3. **Validate password before sending** (UX, not security)
4. **Never store passwords** in browser storage

### Why Not Hash on Frontend?

```javascript
// ❌ WRONG: Hashing in frontend
const hashed = await hashPassword(password); // Bad idea!
fetch('/api/login', { body: JSON.stringify({ password: hashed }) });
```

**Problem:** If the hash is what's sent to the server, then the hash becomes the actual password. Anyone who steals the hash can log in.

### Frontend Best Practices:

```javascript
// ✅ CORRECT: Send plain password over HTTPS
async function login(email, password) {
  const response = await fetch('/api/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }) // Plain text, but over HTTPS
  });
  // Server handles hashing and comparison
}
```

---

## 1-10: Rainbow Table Attack

### What is a Rainbow Table?

A **rainbow table** is a precomputed table of hash → original password mappings.

```
Hash: 5f4dcc3b5aa765d61d8327deb882cf99
Rainbow Table Lookup → "password"
```

### Why Frontend Developers Should Know This:

1. **Understand why salting is critical** (backend, but affects security)
2. **Know why websites ask for password complexity** (prevents brute-force)
3. **Understand password security best practices** to implement proper auth flows

### Protection Against Rainbow Tables (Backend Concern):

- **Salting**: Unique salt per password
- **Strong Algorithms**: bcrypt, Argon2
- **Work Factor**: High computational cost

### Frontend Role:
- Ensure HTTPS for transmission
- Never store or cache passwords
- Implement proper logout and session expiration

---

## 1-11: What is Ciphertext?

### Definition

**Ciphertext** is encrypted plaintext — data that has been transformed to be unreadable without the decryption key.

```
Plaintext + Key → Encryption → Ciphertext
"Hello"  +  [key]  →   AES    → "3x7f9K2m..."
```

### Frontend Context:

**Frontend uses encryption for:**
- HTTPS (TLS encryption — browser handles it)
- End-to-end encryption in messaging apps
- Encrypting data before storing locally
- Client-side encryption for sensitive user data

### Frontend Encryption Example:

```javascript
// Using Web Crypto API for client-side encryption
async function encryptMessage(message, key) {
  const encoder = new TextEncoder();
  const data = encoder.encode(message);
  const ciphertext = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv: new Uint8Array(12) },
    key,
    data
  );
  return ciphertext;
}
```

### Key Points for Frontend:
- HTTPS handles transport encryption automatically
- Client-side encryption is for special use cases
- Never implement custom encryption
- Use Web Crypto API or trusted libraries

---

## 1-12: Interpreted vs. Compiled (Frontend Context)

### Interpreted Languages (JavaScript)

- JavaScript is interpreted/just-in-time compiled
- Source code is visible to users
- Code can be inspected via DevTools

```javascript
// Users can see this!
function authenticate(user, pass) {
  // Never put secrets here!
}
```

### Compiled Languages (C++, Rust)

- Code is compiled to machine code
- Source code is not visible
- Higher performance

### Security Implications for Frontend:

1. **Source code visibility**: Every user can see your JS
2. **No secrets in frontend**: API keys, tokens, passwords
3. **Minification/Obfuscation**: Makes code harder to read but doesn't make it "secure"
4. **Source maps**: Don't expose in production

### Frontend Best Practice:
```javascript
// ❌ NEVER DO THIS IN FRONTEND
const API_KEY = 'sk_live_abc123def456'; // Visible to everyone!
const DB_PASSWORD = 'superSecret123';

// ✅ DO: Use environment variables (still visible in build!)
// Better: Use a backend proxy/BFF pattern
```

---

## 1-13: Stateless vs. Stateful (Frontend Perspective)

### Stateless (Frontend View)

Frontend treats each request as independent.

```
Request 1 → (with token) → Server → Response 1
Request 2 → (with token) → Server → Response 2
```

**Frontend Implementation:**
```javascript
// Stateless authentication
async function apiCall(url) {
  const token = getTokenFromMemory(); // Store token in memory
  const response = await fetch(url, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  // Server doesn't store session; token contains all info
}
```

### Stateful (Frontend View)

Frontend relies on server to maintain session.

```
Request 1 → (with cookie) → Server (stores session) → Response 1
Request 2 → (with cookie) → Server (looks up session) → Response 2
```

**Frontend Implementation:**
```javascript
// Stateful authentication (cookie-based)
async function apiCall(url) {
  const response = await fetch(url, {
    credentials: 'include' // Browser automatically sends cookies
  });
  // Server maintains session in memory/database
}
```

### Which is Better for Frontend?

| Aspect | Stateless (JWT) | Stateful (Session) |
|--------|-----------------|--------------------|
| **Frontend Complexity** | More (manage token) | Less (browser handles cookies) |
| **Security** | Depends on storage | HttpOnly cookies more secure |
| **Logout** | Must clear token manually | Server destroys session |
| **Cross-tab sync** | More complex | Automatic |

---

## 1-14: Login System with JWT (Frontend Implementation)

### JWT Login Flow (Frontend Side)

```
1. User submits credentials
2. Frontend sends to backend via HTTPS
3. Backend returns JWT
4. Frontend stores token securely
5. Frontend includes token in all API requests
6. Frontend handles token expiration
```

### Frontend Implementation:

```javascript
// Login function
async function login(email, password) {
  try {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });
    
    const data = await response.json();
    
    if (data.token) {
      // Store token in memory (most secure)
      window.__token = data.token;
      // Or use httpOnly cookie (set by server)
      // NEVER use localStorage or sessionStorage
      return true;
    }
    return false;
  } catch (error) {
    console.error('Login failed:', error);
    return false;
  }
}

// Authenticated API call
async function fetchProtectedData() {
  const token = window.__token;
  
  if (!token) {
    redirectToLogin();
    return;
  }
  
  const response = await fetch('/api/protected', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });
  
  if (response.status === 401) {
    // Token expired
    redirectToLogin();
    return;
  }
  
  return response.json();
}
```

### Token Storage Options (Frontend):

| Storage Method | XSS Safe | CSRF Safe | Persistence |
|----------------|----------|-----------|-------------|
| **Memory (State)** | ✅ | N/A | ❌ (lost on refresh) |
| **HttpOnly Cookie** | ✅ | ❌ (needs CSRF protection) | ✅ |
| **sessionStorage** | ❌ | ✅ | ✅ (tab session) |
| **localStorage** | ❌ | ✅ | ✅ (persistent) |

### Best Practice:
- **For JWT**: Store in memory only (React state, Vuex, Redux)
- **For cookies**: Use HttpOnly, Secure, SameSite attributes
- Always implement token refresh mechanism

---

## 1-15: Deep Dive into Stateless Server (Frontend Perspective)

### Understanding Stateless Architecture

Stateless means the server doesn't remember anything about previous requests. Each request must contain all necessary information.

### Frontend Impact:

**With Stateless:**
- Frontend stores and sends authentication token
- Token contains user info and expiration
- No server sessions to manage
- Any server can handle any request

### Token Lifecycle (Frontend):

```
1. Login → Receive token (15 min expiry)
2. Use token for requests
3. Before expiry, request refresh token
4. Store new token
5. Repeat
```

### Refresh Token Pattern:

```javascript
// Token refresh interceptor
async function fetchWithAuth(url, options = {}) {
  const token = getToken();
  
  if (isTokenExpired(token)) {
    // Refresh token silently
    const newToken = await refreshToken();
    setToken(newToken);
  }
  
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${getToken()}`
    }
  });
}

async function refreshToken() {
  const response = await fetch('/api/refresh', {
    method: 'POST',
    credentials: 'include' // Refresh token in httpOnly cookie
  });
  const data = await response.json();
  return data.token;
}
```

### Stateless vs. Stateful for Frontend:

| Concern | Stateless | Stateful |
|---------|-----------|----------|
| **Token storage** | Must manage in frontend | Browser manages (cookies) |
| **Cross-tab sync** | Need to sync token | Automatic |
| **Logout** | Clear token + server | Server destroys session |
| **Scalability** | Any server works | Need session sharing |

---

## 1-16: JWT vs. JWE (Frontend Context)

### JWT (JSON Web Token)

- Signed (authenticity verified)
- **Can be decoded** and read by anyone (including frontend)
- Contains: Header, Payload, Signature

**Frontend Access:**
```javascript
// JWT is base64 encoded, but NOT encrypted
function decodeJWT(token) {
  const parts = token.split('.');
  const payload = JSON.parse(atob(parts[1]));
  console.log(payload); // Anyone can read this!
  return payload;
}
```

### JWE (JSON Web Encryption)

- Encrypted (confidential)
- Cannot be read without decryption key
- Contains sensitive data hidden from all parties

### Frontend Usage:

| Scenario | Use |
|----------|-----|
| User authentication | **JWT** |
| User ID, role, permissions | **JWT** |
| Sensitive user data in token | **JWE** |
| Sharing data between services | **JWE** |

### Frontend Best Practice:
```javascript
// NEVER put sensitive data in JWT payload
// ❌ BAD
jwt.sign({
  userId: 123,
  password: 'secret123', // Exposed!
  creditCard: '4111-1111-1111-1111' // Exposed!
});

// ✅ GOOD
jwt.sign({
  userId: 123,
  role: 'admin',
  exp: Date.now() + 900000
});
```

---

## 1-17: Deep Dive into Cookies (Frontend Focus)

### Cookies in Frontend

Cookies are small pieces of data stored in the browser and automatically sent with HTTP requests.

### Frontend Cookie Access:

```javascript
// Reading cookies (non-HttpOnly only)
document.cookie // "theme=dark; sessionId=abc123"

// Setting cookies
document.cookie = 'theme=dark; Secure; SameSite=Lax; Max-Age=86400';
```

### Cookie Attributes (Frontend Must Know):

| Attribute | Frontend Impact |
|-----------|-----------------|
| **HttpOnly** | ❌ Cannot access via JavaScript |
| **Secure** | Only sent over HTTPS |
| **SameSite=Strict** | Not sent on cross-site requests |
| **SameSite=Lax** | Sent on top-level navigation |
| **SameSite=None** | Sent on all cross-site requests (requires Secure) |

### Securing Cookies from Frontend:

```javascript
// ❌ BAD: Accessing auth cookies
const sessionId = document.cookie.split(';').find(c => c.includes('session'));
// HttpOnly cookie returns undefined (good!)

// ✅ GOOD: Only access cookies for non-sensitive data
function getTheme() {
  const match = document.cookie.match(/theme=([^;]+)/);
  return match ? match[1] : 'light';
}
```

### Cookie Security Best Practices:

1. **Authentication cookies** → HttpOnly, Secure, SameSite
2. **Non-sensitive cookies** → Still use Secure and SameSite
3. **Never store personal data** in client-side cookies
4. **Clear cookies on logout** via backend or expire

---

## 1-18: SameSite Cookie Attribute

### What is SameSite?

Controls when cookies are sent in cross-origin requests. **Critical for CSRF protection.**

### Three Values:

**1. Strict (Most Secure)**
```
Cookie: session=123; SameSite=Strict
```
- Cookie not sent on any cross-site requests
- User must be on the same site
- ✅ Best CSRF protection
- ❌ May break legitimate cross-site links

**2. Lax (Recommended for Frontend)**
```
Cookie: session=123; SameSite=Lax
```
- Sent for top-level navigation (user clicks link)
- Not sent for AJAX or form submissions
- ✅ Good CSRF protection
- ✅ Good user experience

**3. None (Use Only When Necessary)**
```
Cookie: session=123; SameSite=None; Secure
```
- Sent on all cross-origin requests
- Must be used with Secure
- ⚠️ CSRF vulnerable
- For third-party integrations only

### Frontend Effect:

```javascript
// With SameSite=Lax
// User clicks link from Twitter → Browser sends session cookie ✅
// JavaScript fetch from Twitter → Browser blocks session cookie ❌

// Frontend fetch
fetch('https://example.com/api/data', {
  credentials: 'include'
  // Cookie sent only if SameSite allows
});
```

### Frontend Best Practice:
- Use `SameSite=Lax` by default
- Only use `None` when absolutely necessary
- Always use `Secure` with `None`
- For cross-origin AJAX, use other auth methods (Bearer tokens)

---

# Summary: Chapter 1 Core Concepts (Frontend Focus)

| Concept | Frontend Key Takeaway |
|---------|----------------------|
| **Origin** | Browser security boundary; affects API requests |
| **Storage** | localStorage/sessionStorage are XSS vulnerable |
| **Cookies** | Use HttpOnly for auth; SameSite for CSRF protection |
| **Hash** | Used for integrity (SRI), not password hashing in frontend |
| **JWT** | Store in memory, not localStorage |
| **Stateless** | Frontend manages tokens; backends scale easier |

---

**Next Chapter:** Chapter 2 — Hands-on with DVWA Attack Labs