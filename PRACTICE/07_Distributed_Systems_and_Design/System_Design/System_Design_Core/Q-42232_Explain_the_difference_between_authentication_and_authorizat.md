---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between authentication and authorization.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Load Balancers](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question "Explain the difference between authentication and authorization" is a basic-concept check. The interviewer wants a crisp, correct definition, the ability to keep the two ideas from blurring, and a couple of concrete examples. Even though it's foundational, strong candidates add the ordering (authN before authZ), where each lives in a real request, and the common HTTP status codes tied to each. I'll define both terms plainly, give an everyday analogy, show them in a real login flow, and note where people go wrong.

The one-line distinction
Authentication (authN) answers "Who are you?" — it verifies identity. You prove you are who you claim: username + password, a one-time code, a fingerprint, a Google login.
Authorization (authZ) answers "What are you allowed to do?" — it verifies permissions. Given a known identity, it decides whether you may perform a specific action or access a specific resource.

Authentication always comes first; authorization comes after. You can't decide what someone is permitted to do until you know who they are. (There is a narrow exception — anonymous public resources need no authN — but for protected actions the order holds.)

An everyday analogy

Think of boarding a flight. At the check-in desk they compare your face to your passport — that's authentication: confirming you are the person you claim to be. Later, your boarding pass says seat 14C in economy. When you try to walk into the business-class cabin, the crew stops you — the boarding pass authorizes you only for economy. Same person (authenticated), but not permitted (not authorized) for that seat. Passport = identity; boarding pass = permissions.

In a real web request
You log in with email + password. The server checks the password hash → authentication succeeds, and you get a session/token proving your identity.
You call DELETE /api/users/42. The server first checks your token is valid (still authN — "is this really Asha?"), then checks whether Asha's role allows deleting users → authorization. If Asha is a normal user, she's authenticated but not authorized, so the action is denied.

This maps cleanly to HTTP status codes, which interviewers love to hear:

401 Unauthorized → actually an authentication failure ("who are you?" — missing/invalid/expired credentials). The name is historically misleading.
403 Forbidden → an authorization failure ("I know who you are, but you're not allowed"). Identity is fine; permissions are not.

Remembering "401 = not logged in, 403 = logged in but not allowed" cements the distinction.

Where each is implemented
Authentication mechanisms: passwords + hashing, multi-factor auth (TOTP, WebAuthn/passkeys), OAuth 2.0 / OpenID Connect social login, SAML SSO, API keys, mutual TLS.
Authorization models: RBAC (role-based — "admins can delete"), ABAC (attribute-based — "editors can edit docs they own in their department"), ACLs (per-resource permission lists), and scopes/claims embedded in a token. OAuth 2.0 scopes are pure authorization; OpenID Connect adds the identity layer (authentication) on top.
Why the distinction matters in practice

Conflating them causes real security bugs. The classic one is Broken Object Level Authorization (BOLA/IDOR) — the app authenticates the user but forgets to check they own the resource, so any logged-in user can fetch /api/invoices/1001 belonging to someone else. The system did authN but skipped authZ. Another: putting authorization logic only in the UI (hiding a button) while the API still lets any authenticated user hit the endpoint — authorization must be enforced server-side on every request, not assumed from a valid login.

A subtle point: authentication is typically a one-time event per session (you log in, get a token), while authorization is per-request/per-action (checked every time you touch a protected resource). So a valid token is necessary but not sufficient — each request still needs a permission check.

Summary — what a strong answer emphasizes

Authentication = verifying identity ("who are you?"); Authorization = verifying permissions ("what can you do?"). AuthN runs first and usually once per session; authZ runs after and on every action. Map them to 401 (authN failure) vs 403 (authZ failure), give a concrete example (login vs. deleting another user's data), and mention that both must be enforced on the server — never trust the client. Common mistakes to avoid saying: using the terms interchangeably, claiming 401 means "no permission" (it means "not authenticated"), or forgetting that a valid token still needs per-request authorization. Follow-up probes: "Give an example where authN passes but authZ fails" (IDOR/wrong role) and "Which HTTP code for each?" (401 vs 403).

**Comparison:**

```
Aspect	Authentication (authN)	Authorization (authZ)
Question answered	"Who are you?"	"What are you allowed to do?"
Purpose	Verify identity	Verify permissions
Order	Runs first	Runs after authN
Frequency	Usually once per session	Every request / action
Example	Login with password + MFA	Role check to delete a record
Failure HTTP code	401 Unauthorized	403 Forbidden
Mechanisms	Passwords, OAuth/OIDC, MFA, SSO	RBAC, ABAC, ACLs, scopes/claims
Visible to user as	Login screen	"Access denied" / hidden action
```

**Code:**

```
JS
Copy
// Express middleware showing the two checks as distinct steps
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  try {
    req.user = verifyJwt(token);          // WHO are you? -> identity
    next();
  } catch {
    return res.status(401).json({ error: 'authentication_required' }); // 401 = authN
  }
}

function authorize(...allowedRoles) {
  return (req, res, next) => {
    // req.user already set by authenticate — identity is known
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: 'forbidden' }); // 403 = authZ
    }
    next();
  };
}

// authN runs first, then authZ:
app.delete('/api/users/:id', authenticate, authorize('admin'), deleteUser);
```

**Example input / output:**

```text
# Authenticated as a normal user (valid token), deleting a user:
DELETE /api/users/42   Authorization: Bearer <valid-token-role=user>
-> 403 Forbidden         # authN passed, authZ failed

# No / expired token:
DELETE /api/users/42   Authorization: Bearer <expired>
-> 401 Unauthorized      # authN failed (never reaches authZ)

# Authenticated as admin:
DELETE /api/users/42   Authorization: Bearer <valid-token-role=admin>
-> 200 OK                # authN + authZ both pass
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Request with credentials/token
6. Authenticated?Who are you?
7. 401 Unauthorized
8. Authorized?Allowed to do this?
9. 403 Forbidden
10. 200 Perform action

