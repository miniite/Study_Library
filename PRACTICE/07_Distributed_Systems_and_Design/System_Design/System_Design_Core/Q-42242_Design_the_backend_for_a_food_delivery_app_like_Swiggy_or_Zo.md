---
type: "practice"
difficulty: "Intermediate"
---

# Design the backend for a food delivery app like Swiggy or Zomato.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Load Balancers](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Design the backend for a food delivery app (Swiggy / Zomato)

#### Triage
— what is really being asked

The interviewer wants a clear service decomposition and the order lifecycle for a food-delivery platform, at "intermediate experience" level — so a practitioner answer: name the services, the datastores, and walk the order flow end to end, with the trade-offs a working engineer should know. Food delivery is a three-sided marketplace: customers, restaurants, and delivery partners. The signature challenge (vs. plain e-commerce) is the three-way real-time coordination of an order between all three parties, plus restaurant discovery/search and live delivery tracking.

Clarifying questions / assumptions:

Scope: browse restaurants → cart → place order → restaurant accepts/prepares → assign delivery partner → live tracking → delivered → pay/rate. Assume city-level scale, food + a courier fleet.

#### Cons
istency: order + payment must be reliable; delivery partner location is best-effort real-time.
Reuse: this shares a lot with ride-hailing (geo-matching, live tracking) — I'll call that out.

Plan: services → data stores → the order lifecycle state machine → search/discovery → delivery matching → payments → async backbone → scale notes → summary.

Core services
API Gateway / BFF — auth (JWT), routing, rate-limiting; holds push connections (WebSocket) for live order status and courier tracking.
User service — customers, addresses, saved payment methods, favourites.
Restaurant service — restaurant profiles, menus, pricing, opening hours, and crucially live availability (open/closed, item in-stock, "kitchen busy" prep-time). Menus are read-heavy → cached hard.
Search / Discovery service — "restaurants near me", cuisine/dish search, filters (rating, veg, price, delivery time), ranking/personalization. Backed by Elasticsearch + a geo filter.
Cart & Order service — the heart. Builds a cart, validates against the current menu/price/availability at checkout, creates the order, and owns the order state machine.
Delivery / Logistics service — assigns a nearby delivery partner (same geo-matching as ride-hailing), tracks the partner from restaurant → customer.
Location service — ingests courier GPS pings into an in-memory geo index (Redis GEO), answers "nearest free partner".
Payments service — pre-auth/charge, wallet, refunds, restaurant & partner payouts; idempotent with a PSP (Razorpay/Stripe).
Pricing service — item total + taxes + delivery fee (distance-based) + surge in bad weather/peak + coupons.
Notification service — push/SMS to all three parties, driven off events.
Data stores (polyglot, match the access pattern)
Data	Store	Why
Users, orders, payments	PostgreSQL (sharded by city)	ACID for money + order state
Restaurants & menus	Postgres + Redis cache	read-heavy, cache aggressively
Restaurant/dish search	Elasticsearch	geo + text + filters + ranking
Courier live location	Redis GEO	ms geo reads, disposable pings
Events / async	Kafka	decouple, replay, notifications
Order history / analytics	warehouse / ES	reporting, "my orders"
The order lifecycle (a state machine — the core of the design)

Food delivery is fundamentally an order state machine coordinating three actors. Transitions are strict and, because money is involved, idempotent and atomic (each state change writes the new state + an outbox event in one DB transaction; a relay publishes to Kafka so payments/notifications/logistics never miss it — the transactional outbox pattern).

PLACED → RESTAURANT_ACCEPTED → PREPARING → READY_FOR_PICKUP → PARTNER_ASSIGNED → PICKED_UP → OUT_FOR_DELIVERY → DELIVERED

with branches: RESTAURANT_REJECTED (auto-refund), CANCELLED_BY_USER (fee rules by stage), and delivery failures. Two coordination points make this harder than e-commerce: the restaurant must accept (it can reject/be too busy) and a delivery partner must be found — often started in parallel with cooking so the courier arrives as the food is ready, minimizing wait.

Search & discovery

This is a heavier feature than in ride-hailing. Checkout is preceded by browsing: "restaurants near me, open now, delivering to my address, sorted by ETA/rating, filtered by cuisine/veg/offers." Implement with Elasticsearch: index restaurants with a geo_point, cuisines, dishes, rating, avg prep time; query = geo-distance filter + text + filters + a ranking that blends distance, rating, delivery time, and personalization. Serviceability (is this restaurant within delivery range of the user's address?) is a geo check done up front. Menus themselves are cached in Redis because they're read on nearly every screen and change infrequently.

Delivery partner matching (reuse ride-hailing)

Once an order is ready (or predicted-ready), the Logistics service finds the best courier: query the Redis GEO index for free partners near the restaurant, rank by ETA-to-restaurant + rating + current load (a partner may batch 2 nearby orders), then run a timed offer loop with an atomic lock to avoid assigning one partner to two conflicting orders. This is exactly the geo-matching + offer pattern from ride-hailing — I'd reuse the design and say so. Batching (assigning one courier multiple orders along a route) is a food-delivery-specific optimization that improves fleet efficiency.

Payments

At checkout, pre-authorize the amount (or charge for prepaid). On DELIVERED, capture. Restaurant rejection or cancellation before cooking → auto-refund. All calls carry idempotency keys because mobile retries are constant; the order-create is idempotent on a client-supplied request id so a double-tap doesn't create two orders. Restaurant and partner payouts are settled asynchronously off the completed-order stream.

Async backbone & why it matters

Kafka decouples everything non-critical. order.placed, order.accepted, partner.assigned, order.delivered events fan out to Notification (push all three parties), Analytics, Payments, and Logistics. If Notification is down, orders still flow and events replay — no lost updates. Live tracking is pushed to the customer over WebSocket from the Location service (courier dot moving on the map), which is eventually consistent and that's acceptable.

Scale & reliability notes (practitioner level)
Read-heavy menus/search: cache menus in Redis, serve search from ES read replicas, CDN for images. Discovery is the highest-QPS surface.
Location write storm: courier pings go to Redis GEO latest-only, never to the SQL DB — same lesson as ride-hailing.
Peak (lunch/dinner): surge on delivery fee to attract partners; queue/backpressure on order acceptance; shard order DB by city so one city's peak doesn't affect others.
Idempotency + outbox everywhere money or three-party state changes, to survive retries and partial failures.
Observability: trace an order by order_id across services; alert on accept latency, partner-assign time, and payment failures.
Summary — what a strong answer emphasizes

Model it as a three-sided marketplace coordinated by an order state machine (with the restaurant-accept and partner-assign steps that plain e-commerce lacks), backed by polyglot storage (SQL for orders/money, Elasticsearch for discovery, Redis GEO for live courier location), decoupled by Kafka + transactional outbox, with live tracking pushed over WebSocket. Explicitly reuse the ride-hailing geo-matching + offer loop for courier assignment, and add the food-specific pieces: rich search/discovery, restaurant availability, parallel cook-and-dispatch, and order batching.

#### Common mistakes
treating it as simple e-commerce and forgetting the restaurant-accept and courier-assign coordination; storing GPS in the main DB; ignoring idempotency on order/payment; under-designing search. Follow-up probes: how discovery ranking/personalization works, how you predict prep time to time the courier's arrival, and how order batching is decided.

**Comparison:**

```
Sub-problem	Approach	Reused from ride-hailing?
Restaurant discovery/search	Elasticsearch geo + text + filters + ranking	No — food-specific
Order coordination	Order state machine + transactional outbox	Same pattern (trip state machine)
Courier assignment	Redis GEO nearest + ETA rank + timed offer loop	Yes — identical to driver matching
Live tracking	Courier GPS → Redis → WebSocket push	Yes
Payments	Pre-auth + capture, idempotency keys, refunds	Yes
Peak load	Surge delivery fee, city-sharded order DB, cached menus	Partly
Data class	Consistency	Store
Orders / payments / order-state	Strong (ACID)	Postgres
Menus / restaurant profile	Read-heavy, cacheable	Postgres + Redis
Discovery/search	Eventually consistent	Elasticsearch
Courier live location	Best-effort real-time	Redis GEO
```

**Code:**

```

Order state machine (allowed transitions)

PLACED ─▶ RESTAURANT_ACCEPTED ─▶ PREPARING ─▶ READY_FOR_PICKUP │ │ │ │ ▼ ▼ │ ▼ CANCELLED_ RESTAURANT_ │ PARTNER_ASSIGNED ─▶ PICKED_UP BY_USER REJECTED (refund) │ ▼ └────▶ OUT_FOR_DELIVERY ─▶ DELIVERED

CODE
Copy
```sql
-- Atomic state change + outbox event in ONE transaction (never miss/dup)
BEGIN;
  UPDATE orders SET state='PARTNER_ASSIGNED', partner_id=$1
    WHERE id=$2 AND state='READY_FOR_PICKUP';         -- guard illegal transition
  INSERT INTO outbox(topic, payload)
    VALUES ('order.partner_assigned',
            jsonb_build_object('orderId',$2,'partnerId',$1));
COMMIT;   -- relay ships outbox rows to Kafka -> logistics + notifications

-- One partner cannot hold two conflicting active pickups (batching handled separately)
CREATE UNIQUE INDEX uniq_active_pickup
  ON deliveries(partner_id) WHERE state IN ('PARTNER_ASSIGNED','PICKED_UP');
JSON
Copy
// Elasticsearch: "restaurants near me, open, veg, sorted by ETA"
{
  "query": { "bool": {
    "filter": [
      { "geo_distance": { "distance": "5km", "location": {"lat":12.96,"lon":77.63} } },
      { "term": { "isOpen": true } },
      { "term": { "vegOnly": true } }
    ]
  }},
  "sort": [ { "_geo_distance": {"location":{"lat":12.96,"lon":77.63}, "order":"asc"} },
            { "rating": "desc" } ]
}
```

**Example input / output:**

```text
Request  POST /v1/orders  {userId, restaurantId, items, addr, idempotencyKey}
   -> 201 { orderId: ord_55021, state: "PLACED", etaMins: 38 }

Flow (pushed to customer over WebSocket):
   PLACED              -> "Waiting for restaurant to accept"
   RESTAURANT_ACCEPTED -> "Order confirmed, preparing"          (cook + find partner in parallel)
   PARTNER_ASSIGNED    -> "Ravi is picking up your order" + live map
   OUT_FOR_DELIVERY    -> live courier dot, ETA 9 min
   DELIVERED           -> receipt, rate restaurant & partner

Restaurant rejects instead:
   RESTAURANT_REJECTED -> auto-refund, notify user
```

**Data:**

```json
// order.placed event on Kafka
{
  "eventId": "evt_7712",              // idempotency / dedupe key
  "type": "order.placed",
  "orderId": "ord_55021",
  "userId": "usr_88",
  "restaurantId": "rest_204",
  "items": [ {"sku":"veg-biryani","qty":1,"price":210},
             {"sku":"raita","qty":1,"price":40} ],
  "deliveryAddr": { "lat": 12.9345, "lng": 77.6100 },
  "bill": { "subtotal": 250, "tax": 12.5, "deliveryFee": 29, "total": 291.5, "currency": "INR" },
  "placedAt": "2026-07-07T13:04:11Z"
}
```

**Flow:**

1. REST + WebSocket
2. GPS pings
3. order.placed / accepted
4. timed offer
5. live dot
6. Customer App
7. API Gateway / BFF
8. Restaurant App
9. Delivery Partner App
10. Location Svc
11. Search/Discovery Svc
12. (Elasticsearch)
13. Restaurant/Menu Svc
14. (Redis cache + Postgres)
15. Order Svc + State Machine
16. (Postgres: orders/payments)
17. (Kafka)
18. Logistics/Delivery Svc
19. (Redis GEO index)
20. Payments Svc
21. (PSP)
22. Notification Svc
23. FCM/APNs/SMS

