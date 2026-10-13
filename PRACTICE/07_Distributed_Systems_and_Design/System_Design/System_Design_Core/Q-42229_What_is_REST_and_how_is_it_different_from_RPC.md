---
type: "practice"
difficulty: "Intermediate"
---

# What is REST and how is it different from RPC?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue vs Synchronous Calls](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Queue_vs_Synchronous_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question is "What is REST, and how is it different from RPC?" At the intermediate level, the interviewer isn't just after a textbook definition — they want to see that I understand REST as an architectural style with constraints, that I can contrast its resource-oriented, uniform-interface model against RPC's action/verb-oriented, procedure-call model, and that I have real-world judgment about when each is the better fit (public API vs. internal microservice, human-readability vs. performance, evolvability vs. tight coupling). I'll define REST properly, define RPC, contrast them across concrete axes, and close with a decision guide. Assumption: we're comparing HTTP-based REST against RPC styles such as JSON-RPC, XML-RPC, and modern gRPC.

What REST actually is

REST (Representational State Transfer) is an architectural style described by Roy Fielding. It is not a protocol and not "JSON over HTTP" — it's a set of constraints. The important ones:

Client–server — separation of concerns; UI and storage evolve independently.
Statelessness — every request carries all context; the server keeps no per-client session. This enables horizontal scaling: any instance can serve any request.
Cacheability — responses declare whether they're cacheable (Cache-Control, ETag), so intermediaries and clients can reuse them.
Uniform interface — the defining constraint. Resources are named by URIs (/users/42), manipulated through a small fixed set of methods (GET/POST/PUT/PATCH/DELETE) with agreed semantics, and represented by media types (JSON/XML). Optionally HATEOAS — responses embed links to next actions.
Layered system — clients can't tell whether they're talking to the origin server or a proxy/gateway/cache.
Code-on-demand (optional).

The mental model is nouns + a fixed verb set: you model your domain as resources, and HTTP methods are the universal operations on them. "Deactivate user 42" becomes PATCH /users/42 {"active": false} or DELETE /users/42/session.

What RPC is

RPC (Remote Procedure Call) makes a call to a remote service look like a local function call: getUser(42), deactivateUser(42), transferMoney(a, b, 100). The mental model is verbs/actions: you expose procedures, and the client invokes them by name with arguments. Concretely:

JSON-RPC / XML-RPC — one endpoint (e.g. POST /rpc), body names the method: {"method": "deactivateUser", "params": [42], "id": 1}. HTTP is just a transport; you almost always POST, and HTTP status is 200 even for logical errors (the error is in the body).
gRPC — the modern, high-performance flavor: you define services and methods in a .proto file (Protocol Buffers), a compiler generates strongly-typed client/server stubs, and it runs over HTTP/2 with binary framing. Supports unary and streaming calls.
The core difference

REST is resource-oriented — you name things and act on them with a uniform set of methods. RPC is action-oriented — you name operations and call them. REST leans on HTTP's own semantics (methods, status codes, caching, content negotiation); classic RPC treats the network as a dumb pipe and puts everything in the payload. gRPC leans on HTTP/2 + a strict schema for speed and codegen.

This drives everything else: REST URLs proliferate as nouns and are self-describing and cacheable; RPC endpoints proliferate as verbs and are usually not cacheable by generic infrastructure. REST is easy to explore with a browser/curl; gRPC needs generated stubs or reflection tooling.

#### Trade-offs
and when to pick each

Pick REST when:

You're building a public/partner API — broad client reach, self-descriptive, works with any HTTP tooling, browser-friendly, easy to document (OpenAPI).
Your domain maps naturally to resources (CRUD-heavy).
You want HTTP-native caching (CDNs, ETag, conditional GET) and evolvability without recompiling clients.

Pick RPC (esp. gRPC) when:

Internal, high-throughput microservice-to-microservice calls where latency and payload size matter — binary Protobuf + HTTP/2 multiplexing beats JSON/HTTP1.1.
You want a strict contract with generated, type-safe stubs across many languages, reducing integration bugs.
Operations don't map cleanly to CRUD (calculateRoute, runInference, transferMoney) — action naming is more natural than contorting them into resources.
You need streaming (bidirectional/server/client streams).

Practical hybrid: REST/GraphQL at the public edge, gRPC between internal services. Many companies use exactly this split, plus a gRPC-gateway to expose REST when needed.

Common misconceptions to correct
"REST = JSON over HTTP" — no; REST is the constraints. You can do REST with XML, and you can do non-RESTful JSON-over-HTTP (RPC-in-disguise, e.g. POST /getUser).
"gRPC can't do the web" — browsers need gRPC-Web because raw gRPC needs HTTP/2 trailer control the browser doesn't expose.
"RPC is old/dead" — gRPC is heavily used internally at scale (Google, and widely in the industry).
Summary — what a strong answer emphasizes

REST is an architectural style built on statelessness, cacheability, and a uniform, resource-oriented interface over standard HTTP. RPC is an integration style that exposes named procedures/actions and calls them remotely, with gRPC being its high-performance, schema-first, HTTP/2 form. The essence: REST = nouns + fixed HTTP verbs, self-describing & cacheable, best for public/CRUD APIs; RPC = verbs/procedures, tighter coupling & higher performance, best for internal high-throughput and streaming services. The interviewer wants the constraints, the noun-vs-verb contrast, and a defensible decision rule.

**Comparison:**

```
Axis	REST	RPC (JSON-RPC / gRPC)
Orientation	Resources (nouns)	Procedures/actions (verbs)
Interface	Uniform: fixed HTTP methods on URIs	Custom method names per service
Transport use	Leverages HTTP methods, status codes, caching	HTTP as transport; often POST only; gRPC uses HTTP/2
Payload	Text (JSON/XML), human-readable	JSON, or binary Protobuf (gRPC) — compact/fast
Caching	Native (ETag, Cache-Control, CDNs)	Generally not cacheable by generic infra
Contract/typing	Loose; OpenAPI optional	Strong, schema-first (.proto) with codegen (gRPC)
Coupling	Looser; evolvable	Tighter; clients often regenerate on change
Discoverability	Browser/curl-friendly, self-describing	Needs stubs/reflection tooling
Streaming	Limited (SSE/websockets bolt-on)	First-class in gRPC (bi-di streams)
Best fit	Public/partner APIs, CRUD domains	Internal high-throughput microservices, streaming
```

**Code:**

```
REST — resource-oriented; the verb IS the HTTP method

GET /users/42 -> 200 {"id":42,"name":"Asha","active":true} PATCH /users/42 -> 200 Body: {"active": false} # deactivate DELETE /users/42/sessions -> 204 # log out

CODE
Copy

```json
// JSON-RPC — action-oriented; one endpoint, method named in the body
// POST /rpc
{ "jsonrpc": "2.0", "method": "deactivateUser", "params": { "id": 42 }, "id": 1 }
// Response (note: HTTP 200 even on logical error; error lives in the body)
{ "jsonrpc": "2.0", "result": { "id": 42, "active": false }, "id": 1 }
PROTOBUF
Copy
// gRPC — schema-first RPC (user.proto); codegen produces typed stubs
syntax = "proto3";
service UserService {
  rpc GetUser (GetUserRequest) returns (User);
  rpc DeactivateUser (DeactivateUserRequest) returns (User);
}
message GetUserRequest        { int64 id = 1; }
message DeactivateUserRequest { int64 id = 1; }
message User { int64 id = 1; string name = 2; bool active = 3; }
```

**Example input / output:**

```text
# Same intent, two styles

REST  — "deactivate user 42"
  Request : PATCH /users/42        {"active": false}
  Response: 200 OK                 {"id":42,"name":"Asha","active":false}

JSON-RPC — "deactivate user 42"
  Request : POST /rpc              {"method":"deactivateUser","params":{"id":42},"id":1}
  Response: 200 OK                 {"result":{"id":42,"active":false},"id":1}

gRPC — "deactivate user 42"
  Call    : stub.DeactivateUser(DeactivateUserRequest{id: 42})
  Return  : User{id:42, name:"Asha", active:false}   (binary Protobuf over HTTP/2)
```

**Flow:**

1. Public / partner API,browser clients, CRUD
2. Internal service-to-service,low latency, manylanguages
3. What are you building?
4. REST over HTTP
5. gRPC / RPC
6. Nouns + HTTP verbs,cacheable, self-describing
7. Named procedures, Protobuf+ HTTP/2, typed stubs,streaming
8. Common hybrid: REST at theedge, gRPC internally

