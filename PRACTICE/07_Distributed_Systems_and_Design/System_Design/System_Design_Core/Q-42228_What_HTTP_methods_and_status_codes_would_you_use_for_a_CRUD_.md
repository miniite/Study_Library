---
type: "practice"
difficulty: "Basic"
---

# What HTTP methods and status codes would you use for a CRUD API?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ URL Shortener](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/URL_Shortener.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to map the four CRUD operations (Create, Read, Update, Delete) onto HTTP methods and to say which HTTP status codes each operation returns on success and on the common failures. This is a basic-concept question, so the interviewer is checking that I know REST/HTTP fundamentals cleanly: that I don't reach for POST for everything, that I understand the semantic properties of methods (safe, idempotent), and that I return meaningful status codes instead of always returning 200 OK. A strong answer connects each method to a status code and explains why.

Clarifying assumptions I'd state out loud: we're building a resource-oriented JSON API over HTTP (e.g. a "users" resource), we follow REST conventions, and the client is a normal web/mobile client. With that, let me build the answer from first principles.

First principles: HTTP verbs carry meaning

HTTP methods are not interchangeable — each has defined semantics:

Safe = the method doesn't change server state (read-only). GET is safe.
Idempotent = making the same request many times has the same effect as making it once. GET, PUT, DELETE, HEAD are idempotent; POST is not.

These two properties drive correct design. Idempotency matters because networks retry: if a client times out and retries a PUT, no harm is done; retrying a POST may create a duplicate.

Mapping CRUD to methods
Create → POST /users. The server assigns the new resource's ID, so the client can't be idempotent about it — two POSTs create two users. (If the client supplies the full URI/ID, PUT /users/{id} can also create, and that path is idempotent.)
Read → GET /users (collection) and GET /users/{id} (single). Safe and idempotent; never changes state; body must not carry side effects.
Update → PUT /users/{id} for a full replacement, PATCH /users/{id} for a partial update. PUT replaces the whole resource (idempotent). PATCH applies a diff/partial change; the spec allows it to be non-idempotent, but well-designed patches usually are.
Delete → DELETE /users/{id}. Idempotent in effect (the resource ends up gone), though the status code may differ on repeat calls.
Status codes per operation

Success (2xx):

200 OK — successful GET, and PUT/PATCH that return the updated body.
201 Created — successful POST that creates a resource. Include a Location header pointing to the new resource (Location: /users/42), and typically the created body.
204 No Content — successful operation with no response body: common for DELETE, and for PUT/PATCH when you don't return the entity.
202 Accepted — request accepted for asynchronous processing (job queued), result not ready yet.

Client errors (4xx):

400 Bad Request — malformed JSON or invalid parameters.
401 Unauthorized — missing/invalid authentication (you aren't logged in).
403 Forbidden — authenticated but not allowed (you can't touch this resource).
404 Not Found — the resource ID doesn't exist (GET/PUT/PATCH/DELETE /users/{id}).
405 Method Not Allowed — verb not supported on that URL (e.g. DELETE /users collection).
409 Conflict — state conflict: duplicate unique field on create, or a version/edit conflict.
422 Unprocessable Entity — syntactically valid JSON but fails business/validation rules.
429 Too Many Requests — rate limit exceeded.

Server errors (5xx):

500 Internal Server Error — unhandled server fault.
503 Service Unavailable — overloaded/down, often with Retry-After.
Nuances an interviewer likes to hear
POST vs PUT for create: use POST when the server generates the ID (returns 201 + Location); use PUT when the client owns the identifier and you want idempotent create-or-replace.
PUT vs PATCH: PUT = replace entire representation (omitted fields get cleared/reset); PATCH = partial. Sending a partial body to PUT is a classic bug because it wipes unspecified fields.
Delete idempotency: first DELETE /users/42 → 204; a second call can return 204 again (idempotent) or 404 if you insist the resource must exist. Both are defensible; be consistent.
Validation split: 400 for "I can't even parse this" vs 422 for "I parsed it but it breaks a rule" (e.g. email already conforms to JSON but is a duplicate → often 409).
Never overload 200: returning 200 with an {"error": ...} body defeats HTTP and breaks caches, proxies, and client libraries that key off the status line.

#### Common mistakes
Using POST for reads or GET with side effects (breaks caching and safety).
Returning 200 for a create instead of 201 + Location.
Using PUT for partial updates and silently nulling fields.
Collapsing all failures into 500 or all successes into 200.
Confusing 401 (not authenticated) with 403 (authenticated, not authorized).
Summary

For a resource like /users: Create = POST → 201, Read = GET → 200, Update = PUT/PATCH → 200/204, Delete = DELETE → 204. Layer standard 4xx codes for client faults (400/401/403/404/409/422) and 5xx for server faults. A strong answer explains method semantics (safe/idempotent), justifies each status code, and calls out the POST vs PUT and PUT vs PATCH distinctions.

**Comparison:**

```
CRUD	HTTP method	Path	Success code	Idempotent?	Safe?	Common failure codes
Create	POST	/users	201 (+ Location)	No	No	400, 409, 422
Read (list)	GET	/users	200	Yes	Yes	401, 403
Read (one)	GET	/users/{id}	200	Yes	Yes	404
Update (full)	PUT	/users/{id}	200 / 204	Yes	No	400, 404, 409, 422
Update (partial)	PATCH	/users/{id}	200 / 204	Usually	No	400, 404, 422
Delete	DELETE	/users/{id}	204	Yes	No	404
```

**Code:**

```
JS
Copy
// Express example: a full CRUD "users" resource with correct methods + codes
import express from "express";
const app = express();
app.use(express.json());

const users = new Map(); // id -> user
let nextId = 1;

// CREATE -> POST -> 201 + Location
app.post("/users", (req, res) => {
  const { name, email } = req.body ?? {};
  if (!name || !email) return res.status(400).json({ error: "name and email required" });
  if ([...users.values()].some(u => u.email === email))
    return res.status(409).json({ error: "email already exists" });
  const id = nextId++;
  const user = { id, name, email };
  users.set(id, user);
  res.status(201).location(`/users/${id}`).json(user);
});

// READ (collection) -> GET -> 200
app.get("/users", (_req, res) => res.status(200).json([...users.values()]));

// READ (single) -> GET -> 200 / 404
app.get("/users/:id", (req, res) => {
  const user = users.get(Number(req.params.id));
  return user ? res.status(200).json(user) : res.status(404).json({ error: "not found" });
});

// UPDATE (full replace) -> PUT -> 200 / 404 (idempotent)
app.put("/users/:id", (req, res) => {
  const id = Number(req.params.id);
  if (!users.has(id)) return res.status(404).json({ error: "not found" });
  const { name, email } = req.body ?? {};
  if (!name || !email) return res.status(422).json({ error: "name and email required" });
  const user = { id, name, email };
  users.set(id, user);
  res.status(200).json(user);
});

// UPDATE (partial) -> PATCH -> 200 / 404
app.patch("/users/:id", (req, res) => {
  const id = Number(req.params.id);
  const existing = users.get(id);
  if (!existing) return res.status(404).json({ error: "not found" });
  const updated = { ...existing, ...req.body, id };
  users.set(id, updated);
  res.status(200).json(updated);
});

// DELETE -> DELETE -> 204 / 404
app.delete("/users/:id", (req, res) => {
  const id = Number(req.params.id);
  if (!users.has(id)) return res.status(404).end();
  users.delete(id);
  res.status(204).end();
});

app.listen(3000);
```

**Example input / output:**

```text
# CREATE
POST /users            Body: {"name":"Asha","email":"asha@x.com"}
201 Created            Location: /users/1
Body: {"id":1,"name":"Asha","email":"asha@x.com"}

# CREATE duplicate email
POST /users            Body: {"name":"A2","email":"asha@x.com"}
409 Conflict           Body: {"error":"email already exists"}

# READ single
GET /users/1
200 OK                 Body: {"id":1,"name":"Asha","email":"asha@x.com"}

GET /users/999
404 Not Found          Body: {"error":"not found"}

# FULL UPDATE
PUT /users/1           Body: {"name":"Asha K","email":"asha.k@x.com"}
200 OK                 Body: {"id":1,"name":"Asha K","email":"asha.k@x.com"}

# PARTIAL UPDATE
PATCH /users/1         Body: {"name":"Asha Kumar"}
200 OK                 Body: {"id":1,"name":"Asha Kumar","email":"asha.k@x.com"}

# DELETE
DELETE /users/1
204 No Content         (empty body)

# DELETE again (already gone)
DELETE /users/1
404 Not Found          (empty body)
```

