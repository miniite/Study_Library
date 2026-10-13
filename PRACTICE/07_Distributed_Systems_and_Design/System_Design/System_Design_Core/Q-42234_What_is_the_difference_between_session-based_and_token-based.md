---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between session-based and token-based authentication?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue vs Synchronous Calls](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Queue_vs_Synchronous_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Explain how session-based (stateful, server-stored) authentication differs from token-based (typically stateless JWT) authentication — how each works end to end, where state lives, and the trade-offs that decide which to use.

What the interviewer is probing: Do you understand where authentication state lives and the consequences of that choice — scaling, logout/revocation, CSRF/XSS, mobile vs browser, and cross-service auth? At the intermediate level they want the mechanics and the judgment, not a textbook one-liner.

Clarifying assumptions: I'll assume a web/mobile app that needs to authenticate users across many requests. "Token-based" here means the common JWT flavor (self-contained, signed). I'll call out that "token" can also mean an opaque token backed by a store, which blurs the line.

Plan: Define the shared problem (HTTP is stateless) → walk the session flow → walk the token flow → compare on the axes that matter → give a real-world recommendation.

The shared problem

HTTP is stateless: the server forgets you after each request. After a user logs in once, every subsequent request must re-prove "I am this authenticated user" without re-sending the password. Both approaches solve this by handing the client a credential it replays on each request. The difference is where the authoritative state lives and how the server trusts the credential.

Session-based authentication (stateful)
User POSTs credentials to /login.
Server verifies, then creates a session record server-side — an entry in memory, Redis, or a DB — keyed by a random, opaque session ID (e.g. sess:9f2a...). The record holds userId, roles, expiry, etc.
Server returns the session ID in a Set-Cookie header (HttpOnly; Secure; SameSite).
The browser automatically sends that cookie on every subsequent request.
On each request the server takes the session ID, looks it up in the store, and if present+valid, treats the request as authenticated.

The cookie itself is just a random pointer — it carries no user data. The server is the source of truth.

Token-based authentication (typically stateless JWT)
User POSTs credentials to /login.
Server verifies, then signs a JWT containing claims (sub, role, exp, iat) with a secret (HMAC) or private key (RSA/ECDSA). Nothing is stored server-side.
Server returns the token to the client (response body or header).
Client stores it (memory, localStorage, or a cookie) and sends it on each request, usually Authorization: Bearer <jwt>.
On each request the server verifies the signature and exp locally — no lookup. If the signature is valid and unexpired, it trusts the claims inside.

The token is the source of truth; the server holds only the signing key.

The core difference in one line

Session = "here's a ticket stub; I'll check my ledger to see who you are." JWT = "here's a tamper-proof badge I signed; I trust it without looking anything up."

Where the trade-offs bite

Scaling / statelessness. Sessions need a shared store (Redis) so any server behind a load balancer can look them up — sticky sessions or a central store. JWTs need no store, so they scale horizontally trivially — a big reason microservices and mobile backends favor them.

Revocation / logout — the JWT Achilles' heel. With sessions, logout = delete the record; the credential dies instantly, and you can force-logout a user or revoke on password change. A stateless JWT is valid until it expires — you can't easily kill it mid-life. Real systems mitigate with short-lived access tokens (5–15 min) + a long-lived refresh token, plus a server-side denylist of revoked IDs — which quietly reintroduces server state, narrowing the "stateless" advantage.

Security surface. Cookie sessions are auto-sent by the browser, so they're exposed to CSRF (mitigated by SameSite + CSRF tokens). JWTs in localStorage avoid CSRF but are exposed to XSS (any injected script can read the token). Best practice for browsers: store even JWTs in HttpOnly cookies, getting XSS protection back but taking on CSRF handling.

Cross-domain / mobile / third-party. Cookies are domain-bound and awkward for native mobile apps and cross-service calls. Bearer tokens travel anywhere via a header, which is why APIs, mobile clients, and SSO/OAuth ecosystems standardize on them.

Payload size. A session cookie is tiny (just an ID). A JWT carries claims, so it's larger (hundreds of bytes to a few KB) and sent on every request — bandwidth overhead at scale.

Real-world recommendation
Classic server-rendered web app, single domain → session cookies. Simple, instant revocation, small footprint.
SPA + mobile + microservices / third-party API → tokens (JWT access + refresh), ideally the access token in an HttpOnly cookie for browsers and Authorization: Bearer for mobile.
Hybrid (common in production): short-lived stateless JWT access tokens for fast, store-free verification, plus stateful refresh tokens in Redis so you keep revocation power. This is the pragmatic sweet spot and what most teams (e.g. Auth0-style setups) actually ship.
Summary / what a strong answer emphasizes

Nail the state location: sessions keep state on the server (opaque ID → lookup), tokens keep state in the signed credential (self-verifying, no lookup). Then show judgment: JWTs win on scale and cross-platform, sessions win on instant revocation and simplicity, and mature systems combine them. Mention that "token" isn't always stateless — an opaque token backed by a store behaves like a session with a header instead of a cookie.

#### Common mistakes
claiming JWTs are "more secure" (they trade one risk class for another); forgetting revocation is hard with JWTs; storing JWTs in localStorage without acknowledging XSS. Follow-up probes: "How do you log a user out of a JWT system?" (denylist + short expiry + refresh rotation), "Where do you store the token in a browser?" (HttpOnly cookie), "How do you rotate the signing key?" (kid header + key set / JWKS).

**Comparison:**

```
Axis	Session-based (stateful cookie)	Token-based (stateless JWT)
Where state lives	Server (Redis/DB), cookie is just an opaque ID	Inside the signed token; server holds only the key
Per-request check	Lookup in store	Local signature + exp verify (no lookup)
Horizontal scaling	Needs shared store / sticky sessions	Trivial — no shared state
Logout / revocation	Instant (delete record)	Hard — valid until exp; needs denylist
Browser attack surface	CSRF (mitigate: SameSite + CSRF token)	XSS if in localStorage; use HttpOnly cookie
Mobile / cross-domain / SSO	Awkward (cookies domain-bound)	Natural (Authorization: Bearer header)
Credential size on wire	Tiny (just an ID)	Larger (carries claims)
Best fit	Server-rendered single-domain apps	SPA + mobile + microservices / APIs
```

**Code:**

```
JS
Copy
// SESSION-BASED (Express + express-session + Redis)
app.post('/login', async (req, res) => {
  const user = await verify(req.body.email, req.body.password);
  if (!user) return res.sendStatus(401);
  req.session.userId = user.id;          // stored server-side (Redis)
  res.sendStatus(200);                    // Set-Cookie: connect.sid=<opaque id>
});
app.get('/me', (req, res) => {
  if (!req.session.userId) return res.sendStatus(401); // lookup in store
  res.json({ id: req.session.userId });
});
app.post('/logout', (req, res) => req.session.destroy(() => res.sendStatus(204))); // instant revoke

// TOKEN-BASED (Express + jsonwebtoken)
const jwt = require('jsonwebtoken');
app.post('/login', async (req, res) => {
  const user = await verify(req.body.email, req.body.password);
  if (!user) return res.sendStatus(401);
  const token = jwt.sign({ sub: user.id, role: user.role }, SECRET, { expiresIn: '15m' });
  res.json({ token });                    // nothing stored server-side
});
function auth(req, res, next) {
  const token = (req.headers.authorization || '').replace('Bearer ', '');
  try { req.user = jwt.verify(token, SECRET); next(); } // local verify, no lookup
  catch { res.sendStatus(401); }
}
app.get('/me', auth, (req, res) => res.json({ id: req.user.sub }));
```

**Example input / output:**

```text
SESSION FLOW
  POST /login {email,pw}          -> 200  Set-Cookie: connect.sid=9f2a...; HttpOnly; Secure; SameSite=Lax
  GET  /me   Cookie: connect.sid  -> server looks up sess:9f2a... in Redis -> 200 {id:"u_123"}
  POST /logout                    -> DEL sess:9f2a... -> next request 401 (revoked instantly)

TOKEN FLOW
  POST /login {email,pw}          -> 200 { token: "eyJhbGci...header.payload.sig" }
  GET  /me   Authorization: Bearer eyJhbGci... -> verify signature+exp locally -> 200 {id:"u_123"}
  (no server-side logout: token stays valid until exp unless it is on a denylist)
```

**Data:**

```text
# Decoded JWT (base64url of header.payload.signature)
HEADER : { "alg": "HS256", "typ": "JWT" }
PAYLOAD: { "sub": "u_123", "role": "admin", "iat": 1712345678, "exp": 1712346578 }
SIGNATURE = HMAC-SHA256( base64url(header) + "." + base64url(payload), SECRET )

# Session record in Redis (server-side source of truth)
KEY  : sess:9f2a7c1e8b...
VALUE: { "userId": "u_123", "role": "admin", "expires": 1712349278 }
```

**Flow:**

1. Session Store (Redis)
2. Server
3. Client
4. Session Store (Redis)
5. Server
6. Client
7. SESSION-BASED
8. TOKEN-BASED (no store)
9. POST /login (creds)
10. store {sid -> userId}
11. Set-Cookie: sid (HttpOnly)
12. GET /me (Cookie: sid)
13. lookup sid
14. userId
15. 200 profile
16. POST /login (creds)
17. JWT (signed, self-contained)
18. GET /me (Authorization: Bearer JWT)
19. verify signature + exp locally
20. 200 profile

