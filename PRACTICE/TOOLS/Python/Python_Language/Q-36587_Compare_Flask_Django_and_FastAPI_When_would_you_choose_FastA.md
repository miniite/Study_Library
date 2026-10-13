---
type: "practice"
difficulty: "Intermediate"
---

# Compare Flask, Django, and FastAPI. When would you choose FastAPI for a new backend service?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Functions Modules and Reuse](<../../../../TOOLS/Python/Python_Language/Functions_Modules_and_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to compare Flask, Django, and FastAPI — three Python web frameworks — and then make a decision: when I would reach for FastAPI on a new backend service. The interviewer is probing two things at once. First, do I actually understand the architectural differences (not just "FastAPI is fast")? Second, can I reason about framework selection like a practitioner — weighing team, timeline, workload, and ecosystem — rather than picking the trendy option? For a mid-level backend/fullstack role, a great answer shows I've shipped on more than one of these and know their sharp edges.

Key clarifying assumptions I'd state up front in a real interview: What kind of service is this — a public JSON API, a server-rendered web app, or an internal microservice? Is it I/O-bound (lots of downstream calls) or CPU-bound? What's the team's Python/async maturity? Greenfield or bolted onto an existing Django monolith? The answer changes with these. I'll cover the general comparison, then give concrete "choose FastAPI when…" criteria.

The three frameworks from first principles

Django is a batteries-included full-stack framework. It ships an ORM, a migrations system, an admin panel, auth, forms, templating, sessions, CSRF protection, and a large "contrib" ecosystem out of the box. Its philosophy is "convention over configuration" — you get a lot for free, but you accept Django's way of doing things. Historically it's synchronous and WSGI-based (async views and an ASGI path exist since Django 3.x/4.x, but the ORM and much of the ecosystem are still sync-first, so async is partial). Django shines when you're building a content-heavy or CRUD-heavy web application — think an internal tool, an e-commerce back office, a CMS — where the built-in admin and ORM save you weeks.

Flask is a microframework. Its core is a thin WSGI layer (routing + request/response) built on Werkzeug and Jinja2. It ships almost nothing else by design — no ORM, no built-in auth — and you assemble the stack from extensions (Flask-SQLAlchemy, Flask-Migrate, Marshmallow, etc.). This gives maximum flexibility and a gentle learning curve, at the cost of you making (and maintaining) more decisions. Flask is also synchronous/WSGI at heart (async support was added but is limited and not its strength). Flask is a great fit for small-to-medium services, prototypes, and teams that want full control over their stack.

FastAPI is a modern, async-first API framework built on Starlette (ASGI) for the web layer and Pydantic for data validation. Its defining feature: you declare request/response shapes with Python type hints, and FastAPI uses them to (a) validate and coerce input, (b) serialize output, and (c) auto-generate interactive OpenAPI/Swagger docs. It's designed natively for async def handlers, so it can hold thousands of concurrent I/O-bound requests on a single worker. FastAPI is optimized for JSON APIs and microservices, not server-rendered HTML apps.

The comparison that matters

The honest framing is not "which is best" but "which fits the workload and team." See the table for a side-by-side, but the load-bearing dimensions are:

Scope: Django = full web app with UI/admin; FastAPI/Flask = API-first.
Concurrency model: FastAPI is async-native (ASGI); Flask/Django are sync-first (WSGI), so under heavy I/O concurrency FastAPI does more with fewer resources.
Validation & docs: FastAPI gives typed validation + auto OpenAPI for free; the others need add-ons (Marshmallow, DRF serializers, drf-spectacular).
Ecosystem maturity: Django is the most mature/opinionated with the deepest ecosystem; Flask is battle-tested and huge; FastAPI is newer but has exploded in adoption.

A subtle but important point: performance differences are mostly about the concurrency model, not raw speed. FastAPI "feels fast" primarily because async I/O lets one process serve many concurrent slow requests (e.g., waiting on a payment gateway or a database). For CPU-bound work, async buys you nothing — the GIL still bites, and you'd scale with processes/workers regardless of framework.

When I'd choose FastAPI for a new backend service

I'd pick FastAPI when most of these hold:

It's an API/microservice, not a rendered web app. JSON in, JSON out, consumed by a React/mobile client or other services. FastAPI has no first-class templating story and doesn't try to be Django.
The workload is I/O-bound and concurrency matters. In a fintech/payments context (Razorpay, PhonePe, CRED — the profile's implied domains) a service that fans out to a gateway, a fraud API, and a DB benefits hugely from async: one worker handles hundreds of in-flight requests while they wait on network. FastAPI + httpx/async drivers exploits this directly.
I want typed contracts and self-documenting APIs. Pydantic models double as the schema, the validator, and the OpenAPI spec. For a service many teams integrate against, the free interactive /docs is a real productivity and correctness win.
The team is comfortable with async and modern typing. This is the honest caveat — async has a learning curve, and one blocking call in an async def handler (a sync DB driver, a time.sleep, a requests.get) stalls the whole event loop. If the team isn't async-fluent, that's a foot-gun.

Conversely, I would NOT choose FastAPI when: I need a full admin/CMS and a mature ORM with migrations today → Django (or Django REST Framework if I want a Django-based API). Or when the service is tiny, sync, and the team already knows Flask → Flask is perfectly fine and lower-ceremony. Or when the workload is CPU-bound (image processing, heavy computation) — async's benefit evaporates and I'd focus on worker scaling in any framework.

#### Common mistakes
& follow-up probes
Mistake: "FastAPI is faster, so always use it." Wrong framing. It's faster for concurrent I/O; it doesn't beat the GIL for CPU work, and misusing async (blocking calls in async handlers) makes it slower than sync Flask.
Mistake: calling Django "not async." Django has ASGI/async views now; the nuance is that its ORM and ecosystem are still largely sync, so async is partial.
Mistake: forgetting FastAPI's lack of a built-in ORM. You still pick SQLAlchemy/SQLModel/Tortoise and wire migrations (Alembic) yourself — it's API-batteries, not full-stack-batteries.
Likely follow-ups: "How do you run FastAPI in production?" (Uvicorn workers behind Gunicorn, ASGI). "How do you avoid blocking the event loop?" (run_in_threadpool / async drivers). "You have a Django monolith — do you rewrite in FastAPI?" (No — add FastAPI as a new service at the edges where async I/O concurrency pays off; don't rewrite working CRUD).
Summary — what a strong answer emphasizes

Frame it as fit, not fashion: Django for full-stack web apps with admin/ORM; Flask for small/flexible sync services; FastAPI for async-first JSON APIs and microservices where typed contracts, auto-docs, and high I/O concurrency matter. Choose FastAPI for a new service when it's an I/O-bound API, the team can handle async, and you value Pydantic validation + OpenAPI — but stay honest that async is a tool for concurrency, not a magic "make it fast" switch, and that one blocking call can undo all of it.

**Comparison:**

```
Dimension	Django	Flask	FastAPI
Type	Full-stack ("batteries included")	Microframework	Async API framework
Server interface	WSGI (ASGI/async partial)	WSGI (limited async)	ASGI (async-native)
Concurrency model	Sync-first	Sync-first	Async-first (async def)
Built-in ORM	Yes (Django ORM + migrations)	No (bring SQLAlchemy)	No (bring SQLAlchemy/SQLModel)
Validation	Forms / DRF serializers	Manual / Marshmallow	Pydantic + type hints (built-in)
Auto API docs (OpenAPI)	Via drf-spectacular	Via add-ons	Built-in (/docs, /redoc)
Admin UI	Yes (killer feature)	No	No
Best for	Web apps, CMS, CRUD + admin	Small/flexible sync services	I/O-bound JSON APIs / microservices
Learning curve	Medium-high (opinionated)	Low	Low-medium (+ async concepts)
Ecosystem maturity	Very high	Very high	High & fast-growing
```

**Code:**

```
The same "create order" endpoint in each framework, to show the ergonomics gap.
---- FastAPI: typed body, auto-validation, async I/O, free OpenAPI docs ----

from fastapi import FastAPI, HTTPException from pydantic import BaseModel, Field import httpx

app = FastAPI()

class OrderIn(BaseModel): amount_paise: int = Field(gt=0) # validated: must be > 0 currency: str = "INR"

class OrderOut(BaseModel): order_id: str status: str

@app.post("/orders", response_model=OrderOut) async def create_order(body: OrderIn): # body is already validated & typed async with httpx.AsyncClient() as client: # non-blocking downstream call r = await client.post("https://gateway/charge", json=body.model_dump()) if r.status_code != 200: raise HTTPException(status_code=502, detail="gateway error") return OrderOut(order_id=r.json()["id"], status="created")

---- Flask: sync, manual validation, no auto-docs ----

from flask import Flask, request, jsonify import requests

flask_app = Flask(name)

@flask_app.post("/orders") def create_order_flask(): data = request.get_json(silent=True) or {} amount = data.get("amount_paise") if not isinstance(amount, int) or amount <= 0: # manual validation return jsonify(error="amount_paise must be a positive int"), 400 r = requests.post("https://gateway/charge", json=data) # blocking call if r.status_code != 200: return jsonify(error="gateway error"), 502 return jsonify(order_id=r.json()["id"], status="created"), 201

---- Django REST Framework: serializer-based (sketch) ----
serializers.py

from rest_framework import serializers class OrderSerializer(serializers.Serializer): amount_paise = serializers.IntegerField(min_value=1) currency = serializers.CharField(default="INR")

views.py

from rest_framework.decorators import api_view from rest_framework.response import Response @api_view(["POST"]) def create_order_django(request): s = OrderSerializer(data=request.data) s.is_valid(raise_exception=True) # DRF validation # ... call gateway (sync), build response ... return Response({"status": "created"}, status=201)

CODE
Copy

```bash
# Production run commands differ by concurrency model:
# FastAPI (ASGI):
gunicorn app:app -k uvicorn.workers.UvicornWorker -w 4
# Flask / Django (WSGI):
gunicorn app:app -w 4            # Flask
gunicorn project.wsgi:application -w 4   # Django
```

**Example input / output:**

```

FastAPI request/response, showing automatic validation:

CODE
Copy
# Valid request
POST /orders
{ "amount_paise": 49900, "currency": "INR" }
-> 200 OK
{ "order_id": "ord_9xK2", "status": "created" }

# Invalid request (negative amount) — FastAPI rejects BEFORE your code runs
POST /orders
{ "amount_paise": -5 }
-> 422 Unprocessable Entity
{
  "detail": [
    { "loc": ["body", "amount_paise"], "msg": "Input should be greater than 0",
      "type": "greater_than" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. No / small & simple
4. Yes
5. Yes
6. No
7. New backend service
8. Server-rendered web appwith admin/CMS/CRUD?
9. Django
10. API / microservice
11. Workload I/O-bound& high concurrency?
12. Flask
13. Team comfortablewith async + typing?
14. FastAPI

