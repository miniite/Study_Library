---
type: "practice"
difficulty: "Deep"
---

# What is WSGI versus ASGI, and why does async support in FastAPI depend on ASGI?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Asyncio Concurrency](<../../../../TOOLS/Python/Python_Language/Asyncio_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question is WSGI vs ASGI, and why FastAPI's async support depends on ASGI. This is a "deep understanding" probe, so the interviewer wants the interface-level story, not slogans. A weak answer says "ASGI is async, WSGI is sync." A strong answer explains what these specifications actually are (contracts between a web server and a Python app), why the WSGI contract structurally cannot support async, what ASGI changed at the protocol level, and the concrete consequences — concurrency model, blocking pitfalls, deployment, WebSockets/streaming. I'll build from first principles: why a specification exists at all, then the shape of each callable, then why async needs the ASGI shape, then production and failure-mode judgment.

Clarifying frame: WSGI and ASGI are not servers or frameworks — they're calling conventions. WSGI is used by Gunicorn/uWSGI ↔ Flask/Django(sync). ASGI is used by Uvicorn/Hypercorn ↔ FastAPI/Starlette/Django(async). FastAPI is built on Starlette, which is ASGI-native; that's the crux.

Why a specification exists at all

Historically, a Python web app had to know how to talk to its specific web server (mod_python, etc.), which coupled apps to servers. WSGI (PEP 3333) solved this by defining a single, minimal contract: any WSGI server can run any WSGI app. This decoupling is why you can put Flask behind Gunicorn, uWSGI, or mod_wsgi without changing app code. ASGI is the successor that preserves this "any server ↔ any app" decoupling while adding capabilities WSGI structurally lacks.

WSGI: a synchronous, one-shot callable

The WSGI contract is a plain callable:

PYTHON
def app(environ, start_response):
    start_response("200 OK", [("Content-Type", "text/plain")])
    return [b"hello"]

Three defining properties:

environ is a dict holding the entire request (method, path, headers, and the request body as a readable stream) — the request is fully framed by the time your app is called.
start_response is called once to set status + headers; the app then returns an iterable of bytes for the body.
It is a normal, synchronous function call. The server calls app(...), the call runs to completion, returns the body, done. There is exactly one request per call, and the call blocks its worker until it finishes.

This last point is the whole limitation. WSGI has no mechanism to yield control back to the server mid-request while waiting on I/O. If your handler makes a database or HTTP call, the OS thread/process is parked doing nothing until the response comes back. To serve N concurrent slow requests, WSGI servers must run ~N workers (processes or threads). That's fine for CPU-bound or fast requests, but memory-expensive and it caps concurrency for I/O-heavy workloads. Crucially, there is no place in the WSGI signature for await — the callable isn't a coroutine, and the server never re-enters it. You cannot bolt async onto this shape without changing the contract.

ASGI: an async, event-driven callable with a message protocol

ASGI redefines the contract to be async-native and event-based. An ASGI app is an async callable with a different signature:

PYTHON
async def app(scope, receive, send):
    assert scope["type"] == "http"
    await send({"type": "http.response.start", "status": 200,
                "headers": [(b"content-type", b"text/plain")]})
    await send({"type": "http.response.body", "body": b"hello"})

The differences are architectural, not cosmetic:

scope describes the connection (type: http, websocket, or lifespan; path; headers) — but not the body. This decoupling of connection metadata from the data stream is what enables long-lived connections.
receive and send are async functions that exchange event messages with the server. The app await receive()s incoming events (request body chunks, WebSocket frames, disconnects) and await send()s outgoing events (response start, body chunks). This is a bidirectional, streaming, message-passing interface — not a single return value.
The app is a coroutine. Because it's async and communicates via await, when it hits await receive() or await some_db_call(), it yields control back to the event loop, which can advance other in-flight requests on the same worker. One worker, one thread, one event loop — thousands of concurrent I/O-bound requests interleaved cooperatively.

Two capabilities fall out naturally that WSGI simply cannot express: (a) WebSockets and long-lived streaming (the scope["type"] == "websocket" path with continuous receive/send), and (b) lifespan events (startup/shutdown hooks). WSGI's "call → return bytes → done" shape has no room for a connection that stays open exchanging messages.

Why FastAPI's async depends on ASGI

FastAPI is a thin layer over Starlette, and Starlette is an ASGI application — its whole design is the async def app(scope, receive, send) contract. When you write async def create_order(...), FastAPI runs that coroutine on the event loop provided by the ASGI server (Uvicorn/Hypercorn). The awaits inside your handler are what let the event loop switch to other requests during I/O waits.

This is impossible under WSGI for a fundamental reason: WSGI has no event loop and no re-entry point. A WSGI server calls your app once and blocks until it returns bytes; there is no await, no yielding, no receive/send channel to resume a suspended coroutine. So even if you wrote async def handlers, a WSGI server couldn't drive them concurrently — it would have to run each coroutine to completion synchronously (e.g., via asyncio.run), losing the entire concurrency benefit. Async concurrency requires a runtime that can suspend and resume many coroutines cooperatively; ASGI is the contract that gives the server that runtime, and WSGI is not. Hence: no ASGI, no real async in FastAPI.

The sharp edge: blocking the event loop

The deep-understanding trap is that ASGI's benefit is cooperative — it depends on your code actually yielding. If inside an async def handler you make a blocking call (a synchronous DB driver, requests.get, time.sleep, heavy CPU work), you block the single event-loop thread, and every concurrent request on that worker stalls until it returns. This is worse than sync Flask, which at least had multiple worker threads. The disciplines:

Use async libraries for I/O in async handlers: httpx.AsyncClient (not requests), async DB drivers (asyncpg, async SQLAlchemy), asyncio.sleep.
For unavoidable blocking/CPU work, offload it: FastAPI runs a def (non-async) handler in a threadpool automatically, or you use run_in_threadpool / loop.run_in_executor, or a separate worker (Celery/RQ) for CPU-bound jobs.
Remember async does not beat the GIL for CPU-bound work — it's a concurrency tool for I/O, not parallelism for computation. Scale CPU work with processes/workers.
Deployment consequences

You must run FastAPI on an ASGI server — commonly Uvicorn (workers), often supervised by Gunicorn: gunicorn app:app -k uvicorn.workers.UvicornWorker -w 4. Running it under a plain WSGI server won't work. Django, which straddles both worlds, offers wsgi.py and asgi.py; to use its async views you deploy the ASGI entrypoint. Rule of thumb for scaling: workers ≈ CPU cores for parallelism, while each async worker handles many concurrent I/O-bound requests via the loop — so a small number of ASGI workers can serve very high I/O concurrency, which is exactly the win in fintech/foodtech services that fan out to many downstream APIs.

#### Common mistakes
& follow-ups
Mistake: "ASGI is just async WSGI." No — it adds a message protocol (receive/send events, scope types) enabling WebSockets, streaming, and lifespan, not merely async.
Mistake: assuming async def alone makes it concurrent. Concurrency comes from the ASGI event loop plus non-blocking code; blocking calls negate it.
Mistake: "async is faster." It raises concurrency/throughput for I/O-bound work; latency of a single request isn't magically lower, and CPU work doesn't benefit.

#### Follow-ups
"Draw the ASGI callable signature." "What happens if you time.sleep(5) in an async handler under load?" (whole worker stalls). "How does Django support both?" (WSGI + ASGI entrypoints; ORM still largely sync). "Why 422 vs streaming — where does ASGI help?" (chunked http.response.body events, Server-Sent Events, WebSockets).
Summary

WSGI (PEP 3333) is a synchronous server↔app contract: a single blocking callable, one request per call, environ in and a bytes-iterable out — with no way to suspend a request during I/O, so concurrency costs one worker per in-flight request and async/await has nowhere to live. ASGI is an async, event-driven successor: async def app(scope, receive, send), a bidirectional message protocol over an event loop, enabling suspend/resume of many coroutines, plus WebSockets, streaming, and lifespan. FastAPI is built on Starlette, an ASGI app; its async def handlers run on the ASGI server's event loop, and that loop's ability to yield on await and resume is exactly what WSGI lacks — so FastAPI's async is only possible because of ASGI. The senior nuance: the benefit is cooperative, so a single blocking call sinks the whole worker, and async buys I/O concurrency, never CPU parallelism.

**Comparison:**

```
Aspect	WSGI (PEP 3333)	ASGI
Callable shape	app(environ, start_response) (sync)	async app(scope, receive, send)
Execution model	Synchronous, blocking, one req/call	Async, event-loop, coroutines suspend/resume
Request data	Full request in environ dict	scope (conn meta) + streamed via receive
Response	Return iterable of bytes	send events (response.start, response.body)
Concurrency for I/O	1 in-flight per worker → many workers	Many in-flight per worker (one loop)
WebSockets / streaming	No	Yes (websocket scope, chunked body events)
Lifespan (startup/shutdown)	No	Yes (lifespan scope)
Servers	Gunicorn, uWSGI, mod_wsgi	Uvicorn, Hypercorn, Daphne
Frameworks	Flask, Django (sync)	FastAPI, Starlette, Django (async)
Supports async def handlers	No (no event loop / re-entry)	Yes (its whole reason to exist)
```

**Code:**

```
WSGI app (PEP 3333): synchronous, one call -> return bytes, no await, no yielding.

def wsgi_app(environ, start_response): # entire request already framed in environ; the call BLOCKS its worker body = do_blocking_db_read() # worker parked during I/O start_response("200 OK", [("Content-Type", "text/plain")]) return [body] # single bytes iterable, then done

ASGI app: async callable, event-based receive/send, yields to the loop on await.

async def asgi_app(scope, receive, send): assert scope["type"] == "http" body = await do_async_db_read() # await -> loop runs OTHER requests await send({"type": "http.response.start", "status": 200, "headers": [(b"content-type", b"text/plain")]}) await send({"type": "http.response.body", "body": body})

FastAPI (Starlette/ASGI): your coroutine runs on the ASGI event loop.

from fastapi import FastAPI from fastapi.concurrency import run_in_threadpool import httpx

app = FastAPI()

@app.get("/good") async def good(): async with httpx.AsyncClient() as c: # non-blocking -> stays concurrent r = await c.get("https://svc/health") return r.json()

@app.get("/danger") async def danger(): import time; time.sleep(3) # BLOCKS the event loop -> ALL return {"ok": True} # concurrent requests stall!

@app.get("/fixed") async def fixed(): # offload unavoidable blocking work to a threadpool -> loop stays free return await run_in_threadpool(some_blocking_sync_call)

CODE
Copy

```bash
# FastAPI must run on an ASGI server:
gunicorn app:app -k uvicorn.workers.UvicornWorker -w 4   # Gunicorn + Uvicorn workers
uvicorn app:app --workers 4                              # or Uvicorn directly
# WSGI apps (Flask/Django-sync) run on a WSGI server:
gunicorn app:app -w 4
gunicorn project.wsgi:application -w 4
```

**Example input / output:**

```

Behavior under 100 concurrent slow (2s I/O) requests, 1 worker — the observable difference:

CODE
Copy
WSGI (1 sync worker):
  requests handled concurrently: 1   (each blocks the worker)
  wall time for 100 reqs:        ~200 s   (serialized)  -> need ~100 workers to parallelize

ASGI (1 async worker, non-blocking I/O):
  requests handled concurrently: ~100 (interleaved on the event loop)
  wall time for 100 reqs:        ~2 s

ASGI (1 async worker) but handler calls time.sleep(2) (BLOCKING):
  requests handled concurrently: 1   (loop is blocked)
  wall time for 100 reqs:        ~200 s   <- async benefit destroyed by blocking call
```

**Flow:**

1. ASGI: async event loop
2. await app(scope, receive,send)
3. receive/send events
4. await yields
5. interleave
6. runs on
7. WSGI: synchronous
8. app(environ, start_response)
9. return [bytes]
10. WSGI serverGunicorn
11. App callBLOCKS worker
12. ASGI serverUvicorn
13. (Event loop)
14. coro req A
15. coro req B
16. coro req C
17. FastAPI/Starlette handlerasync def

