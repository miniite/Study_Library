---
type: "practice"
difficulty: "Intermediate"
---

# How does JWT-based authentication work and where do you store the token?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ URL Shortener](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/URL_Shortener.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

"How does JWT-based authentication work and where do you store the token?" has two parts the interviewer explicitly wants: (1) the mechanics of JWT — its structure, how it's signed and verified, and the stateless request flow — and (2) a judgment call on client-side storage, which is where most candidates get burned. At an intermediate level they expect me to explain the three JWT segments, the difference between signing and encryption, why JWTs are hard to revoke, and to give a defensible storage recommendation (HttpOnly cookie vs. localStorage) with the security reasoning. I'll assume a standard web SPA + backend API, possibly with a mobile client.

What a JWT is

A JSON Web Token (RFC 7519) is a compact, URL-safe string with three Base64URL-encoded parts separated by dots: header.payload.signature.

Header — metadata: the signing algorithm and token type, e.g. {"alg":"RS256","typ":"JWT"}.
Payload (claims) — JSON data about the user/session: registered claims like sub (subject/user id), exp (expiry), iat (issued-at), iss (issuer), aud (audience), plus custom claims like role.
Signature — computed over base64url(header) + "." + base64url(payload) using a secret (HMAC/HS256) or a private key (RSA/ECDSA — RS256/ES256). The signature is what makes the token tamper-evident.

Crucial point: a JWT is signed, not encrypted (unless you use JWE). The payload is only Base64 — anyone can decode and read it. So never put secrets (passwords, card numbers) in a JWT. Signing guarantees integrity and authenticity (nobody altered it, and a trusted party issued it), not confidentiality.

How authentication works with a JWT
Login: client sends credentials to the auth server. On success, the server builds the claims, signs them, and returns the JWT.
Client stores the token (see storage section).
Each request: client sends the token, typically in the header Authorization: Bearer <jwt>.
Verification (stateless): the server (or any microservice) recomputes the signature over header+payload with the shared secret / issuer's public key and checks it matches. It then validates claims: exp not passed, iss/aud expected, etc. If all pass, the request is trusted — no database or session-store lookup needed. That statelessness is JWT's headline benefit: it scales horizontally and lets independent services validate tokens without a shared session store.

With asymmetric signing (RS256/ES256), the auth server holds the private key and every resource service verifies with the public key (often fetched from a JWKS endpoint). Services can validate but never mint tokens — a good security boundary. HS256 (shared secret) is simpler but every verifier must hold the secret, so a leak anywhere lets an attacker forge tokens.

The revocation problem (must-mention)

Because verification is self-contained, a valid, unexpired JWT is accepted even after you "log out" or ban the user — there's nothing central to invalidate. Standard mitigations:

Short access-token TTL (5–15 min) so the exposure window is tiny.
A long-lived refresh token (opaque, stored server-side, revocable) to mint new access tokens; revoking it stops future refreshes.
A denylist/blocklist of jti (token id) or session id in Redis for instant kill, checked cheaply per request — this trades a little of the statelessness back for revocability.
Where do you store the token? (the key judgment)

Two browser options, each with a distinct attack surface:

localStorage / sessionStorage: easy, works cross-domain, survives reloads — but fully readable by JavaScript, so any XSS vulnerability lets an attacker exfiltrate the token. This is the most common insecure choice.
HttpOnly, Secure, SameSite cookie: the browser attaches it automatically; HttpOnly means JavaScript cannot read it, neutralizing token theft via XSS. Secure forces HTTPS. SameSite=Lax/Strict blocks the cookie on cross-site requests, mitigating CSRF. The residual CSRF risk (cookies auto-sent) is handled by SameSite plus a CSRF token / double-submit pattern for state-changing requests.

Recommended pattern for a web SPA: keep the short-lived access token in memory (a JS variable — gone on refresh, minimal XSS exposure, sent as a Bearer header), and keep the refresh token in an HttpOnly Secure SameSite cookie scoped to the /refresh path. On page reload you silently call /refresh to get a new access token. This balances XSS resistance (nothing long-lived in JS-reachable storage) with usability. For mobile apps, store tokens in the OS secure store — Keychain (iOS) / Keystore (Android) — never in plaintext prefs.

Whatever the storage, always transmit over TLS, and never log tokens.

Summary — what a strong answer emphasizes

A JWT is a signed header.payload.signature string; the server verifies the signature + claims locally, so auth is stateless and scales — but the token is only Base64-encoded (readable), so it's signed not secret, and it's hard to revoke (fix with short TTL + revocable refresh token + optional denylist). For storage, avoid localStorage (XSS); prefer access token in memory + refresh token in an HttpOnly/Secure/SameSite cookie for web, and the secure keystore for mobile. Common mistakes: putting sensitive data in the payload, using alg:none or accepting client-chosen algorithms, long-lived non-revocable tokens, and storing in localStorage. Follow-up probes: "How do you revoke a JWT?" (short TTL/denylist), "HS256 vs RS256?" (shared secret vs public/private, verifier trust boundary), and "Why is alg:none dangerous?" (unsigned token accepted → forgery).

**Comparison:**

```
Where to store JWT	XSS exposure	CSRF exposure	Verdict
localStorage / sessionStorage	High — JS can read it	Low	Avoid for long-lived tokens
In-memory JS variable	Low (lost on reload)	Low	Good for short access token
HttpOnly + Secure + SameSite cookie	None (JS can't read)	Mitigated by SameSite + CSRF token	Best for refresh token
Mobile Keychain/Keystore	OS-protected	n/a	Standard for mobile
Signing	Key model	Verifier holds	Trade-off
HS256 (HMAC)	One shared secret	The secret	Simple, but any verifier can forge
RS256/ES256	Private issues, public verifies	Public key only	Verifiers can't mint tokens (safer at scale)
```

**Code:**

```

// Issue and verify a JWT (Node, jsonwebtoken) import jwt from 'jsonwebtoken';

// --- issue on login (asymmetric RS256) --- const token = jwt.sign( { sub: user.id, role: user.role }, // custom claims PRIVATE_KEY, { algorithm: 'RS256', issuer: 'auth.myapp', audience: 'web', expiresIn: '15m' } // exp/iat/iss/aud auto-added );

// --- verify on each request (any service, with public key) --- function authMiddleware(req, res, next) { const token = req.headers.authorization?.replace('Bearer ', ''); try { // ALWAYS pin the algorithm — never trust the token's own header alg req.user = jwt.verify(token, PUBLIC_KEY, { algorithms: ['RS256'], issuer: 'auth.myapp', audience: 'web', }); next(); } catch (e) { return res.status(401).json({ error: 'invalid_or_expired_token' }); } }

Set refresh token as a safe cookie (server response):

Set-Cookie: rt=<opaque>; HttpOnly; Secure; SameSite=Strict; Path=/auth/refresh; Max-Age=2592000
```

**Example input / output:**

```text
POST /login  { "email":"asha@x.com", "password":"S3cret!" }
-> 200 { "accessToken": "eyJhbGciOiJSUzI1Ni...", "expiresIn": 900 }
        Set-Cookie: rt=9f3c...; HttpOnly; Secure; SameSite=Strict

GET /api/profile   Authorization: Bearer eyJhbGciOiJSUzI1Ni...
-> server verifies signature + exp locally (no DB) -> 200 { profile }

# token expired after 15 min:
GET /api/profile   Authorization: Bearer <expired>
-> 401 { "error":"invalid_or_expired_token" }
POST /auth/refresh (cookie rt sent automatically)
-> 200 { "accessToken":"<new>", "expiresIn":900 }

# tampered payload (role changed to admin) -> signature mismatch:
GET /api/admin  Authorization: Bearer <tampered>
-> 401 (signature verification fails)
```

**Data:**

```

Decoded JWT (three parts):

JSON
Copy
// header
{ "alg": "RS256", "typ": "JWT" }
// payload (Base64 only — readable, so NO secrets here)
{
  "sub": "user_42",
  "role": "user",
  "iss": "auth.myapp",
  "aud": "web",
  "iat": 1751856000,
  "exp": 1751856900        // 15 min later
}
// signature: RS256( base64url(header) + "." + base64url(payload), PRIVATE_KEY )

Encoded form (abbreviated):

CODE
Copy
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyXzQyIiwicm9sZSI6InVzZXIi...  .  Qa7c3f...signature
```

**Flow:**

1. API Service
2. Auth Server
3. Client
4. API Service
5. Auth Server
6. Client
7. store access in memory
8. on expiry -> 401 -> POST /refresh -> new JWT
9. POST /login (credentials)
10. verify creds, sign JWT (RS256, exp 15m)
11. accessToken (+ refresh cookie)
12. GET /resource (Authorization: Bearer JWT)
13. verify signature + exp + iss/aud (no DB)
14. 200 data

