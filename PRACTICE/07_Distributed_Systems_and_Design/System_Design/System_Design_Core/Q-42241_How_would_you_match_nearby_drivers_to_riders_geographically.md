---
type: "practice"
difficulty: "Deep"
---

# How would you match nearby drivers to riders geographically?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rate Limiter Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

How would you match nearby drivers to riders geographically?

#### Triage
— what is really being asked

This is the deep dive into the hardest sub-problem of a ride-hailing/delivery backend: given a rider at (lat, lng), find the best few drivers among millions of moving points, in single-digit milliseconds, at hundreds of thousands of location updates per second, and then run a reliable offer/accept loop without double-booking. The interviewer is probing: do I know that a naive "compute distance to every driver" is O(N) and impossible at scale, and do I know the right spatial indexing (geohash / quadtree / S2 / H3 / Redis GEO) plus the operational realities (ranking, offer timeouts, races)?

Clarifying questions / assumptions:

"Nearby" means small radius (1–5 km typical); result set is small (top-K, K≈5–10).
Drivers stream GPS every 3–5s; the index must handle high write churn, latest-only.
"Best" ≠ "closest by straight line" — it means lowest ETA on the road network, adjusted by rating/acceptance/idle-time. I'll separate candidate retrieval (cheap, spatial) from ranking (fewer candidates, richer).
Correctness constraint: one driver → one active offer/trip.

Plan: why brute force fails → spatial index options → the query path → ranking → the offer loop and race handling → hotspots/scale → summary.

Why brute force fails, and the core idea

Distance-to-all-drivers is O(N) per request — with 1M drivers and thousands of requests/sec, it melts. The fix is a spatial index that lets us look up only drivers in the cells around the rider, turning an O(N) scan into an O(k) lookup of a few nearby buckets. Everything hangs on the map being partitioned into cells so "who is near me" becomes "read a handful of cells."

Spatial indexing options
Geohash — encode (lat,lng) into a base-32 string; a shared prefix ≈ spatial proximity. Longer prefix = smaller cell. To query, take the rider's geohash + its 8 neighbours (to avoid edge misses) and read those buckets. Trivial to shard in Redis (key per geohash cell). This is exactly what Redis GEO (GEOADD/GEOSEARCH) does under the hood (sorted sets on a 52-bit geohash score).
Quadtree — recursively split space into 4 quadrants until each leaf has ≤ threshold points; dense areas (downtown) get deeper subdivisions automatically. Great for skewed density; more work to keep balanced under heavy movement.
S2 (Google) / H3 (Uber) — hierarchical cell systems. S2 maps the sphere to 64-bit cell ids on a Hilbert curve (good range scans, no pole/longitude distortion). H3 is hexagonal — every neighbour is equidistant (no diagonal edge problem geohash has), which makes ring/kRing queries and surge cells cleaner. Uber uses H3 in production for exactly this.

Choice: for most systems, Redis GEO / geohash is the pragmatic default (in-memory, ms latency, easy sharding). At Uber-scale or when you also need clean surge/analytics cells, H3 wins. I'd start with Redis GEO and note H3 as the scale evolution.

The query path (retrieval → ranking, two stages)
Retrieve candidates (cheap, spatial): GEOSEARCH a 2–5 km radius around pickup → say 40 online drivers. This is the only step touching the whole population, and it's O(cells), not O(drivers).
Filter: drop drivers who are offline, on a trip, wrong vehicle class, or already have a live offer.
Rank (richer, small set): the ~40 shrink to the top-K by a score, not raw distance:
ETA to pickup on the road network (call Maps/routing, or a fast Haversine + traffic factor as a proxy) — this is the dominant term; a driver 200 m away across a river is worse than one 800 m down the road.
Driver rating, historical acceptance rate, idle time (fairness: reward drivers waiting longest), vehicle match.
Offer loop: send the ride to the #1 driver with a ~10–15s accept timer, pushed over their WebSocket. On accept → assign. On decline/timeout → fall through to #2 (or batch-offer to top few and take first accept). Once someone accepts, cancel the other offers.
Race conditions & correctness (the part juniors miss)

Two riders can be matched to the same free driver at the same instant. Guards:

Atomic reservation: when offering, atomically mark the driver "offered/locked" (Redis SET NX with TTL, or a DB row lock). A second matcher sees the lock and skips that driver.
DB-unique constraint on (driver_id) for active trips, so even if two paths race to assign, the second insert fails and we retry the next candidate.
Offer TTL so a driver who ignores an offer is auto-freed and the offer falls through — no dead locks.
Idempotent accept: driver double-taps accept → same trip, not two.
Hotspots, movement, and scale
Density skew: airports/stations get 1000s of drivers in one cell. Geohash buckets can grow huge → shard those cells further (finer geohash precision) or use quadtree/H3 depth so no single bucket is a scan bottleneck.
Movement churn: drivers move constantly, so index entries update every few seconds. Redis GEO handles this as latest-only GEOADD (overwrite) — cheap, no history.
Sharding: partition the whole index by city/region, so Bangalore's index and Delhi's are separate nodes; a rider only ever queries their region's shard. Horizontal scale = add regions.
Expiry: drivers who stop pinging (app killed, no signal) must age out — TTL / heartbeat sweep so we never offer to a ghost.
Latency budget: retrieval (Redis) ~1–3 ms; ETA calls for ~40 candidates parallelized/cached; total match decision well under ~100 ms so the rider sees "finding your ride" briefly.
Summary — what a strong answer emphasizes

The winning framing is two-stage: cheap spatial retrieval (geohash/H3/Redis GEO) then rich ranking on a tiny candidate set, ranked by ETA (not straight-line distance) plus rating/acceptance/fairness, delivered via a timed offer loop with atomic driver-locking to prevent double-booking. Name the index by trade-off (geohash = simple + Redis-native; H3 = hexagonal, equidistant neighbours, Uber-proven; quadtree = adapts to density). Show you know the operational realities: hotspots, ping expiry, region sharding, and the accept-timeout race.

#### Common mistakes
using Haversine distance as the final rank (ignores roads), scanning all drivers, ignoring the double-offer race, and forgetting stale-driver expiry. Follow-up probes: "why hexagons?" (uniform neighbour distance, no diagonal ambiguity), "how do you compute ETA cheaply?" (precomputed tiles/traffic + routing service, cache per cell), and "how does this tie into surge?" (same cells drive supply/demand multipliers).

**Comparison:**

```
Index	Cell shape	Strengths	Weaknesses	Used by
Geohash / Redis GEO	rectangular	dead simple, Redis-native, ms lookups	edge/neighbour handling, density skew	many teams (default)
Quadtree	adaptive squares	auto-adapts to density	rebalancing under movement	mapping systems
S2	curved quads (Hilbert)	64-bit ids, great range scans, no pole distortion	more complex	Google
H3	hexagonal	uniform neighbour distance, clean rings/surge	slightly heavier	Uber
Stage	Cost	What it does
Retrieval (spatial)	O(cells), ~1–3 ms	narrow millions → ~40 candidates via cells
Filter	O(candidates)	drop offline/on-trip/wrong-class/locked
Rank (ETA + signals)	O(K) with maps calls	pick top-K by road ETA, rating, fairness
Offer loop	timed, sequential/batched	atomic lock, timeout fall-through
```

**Code:**

```
Stage 1: cheap spatial retrieval with Redis GEO (per-city index)

GEOADD drivers:blr 77.6387 12.9611 driver_57 # upsert live pos (latest-only) GEOSEARCH drivers:blr FROMLONLAT 77.6350 12.9600 BYRADIUS 3 km ASC COUNT 40 WITHDIST

-> up to 40 nearest online drivers, sorted by straight-line distance
CODE
Copy
```python
# Stage 2: filter + rank the small candidate set by ETA-based score
def rank_candidates(candidates, pickup):
    scored = []
    for d in candidates:
        if not d.online or d.on_trip or d.has_open_offer:
            continue
        eta   = maps.eta_seconds(d.pos, pickup)          # road ETA, not haversine
        score = (eta                                     # lower is better
                 - d.rating * 30                         # good drivers boosted
                 - d.idle_seconds * 0.1                  # fairness: waited longest
                 + (0 if d.vehicle == req.vehicle else 1e9))
        scored.append((score, d))
    return [d for _, d in sorted(scored)][:5]            # top-K

# Atomic reservation to avoid double-booking (Redis SET NX + TTL)
def offer(driver_id, trip_id, ttl=15):
    got = redis.set(f"lock:driver:{driver_id}", trip_id, nx=True, ex=ttl)
    if not got:
        return False           # already offered/assigned -> skip to next candidate
    push_ws(driver_id, {"type": "OFFER", "tripId": trip_id, "expiresIn": ttl})
    return True
```

**Example input / output:**

```text
Input : rider pickup (12.9600, 77.6350), vehicle=sedan
Stage1: GEOSEARCH 3km -> 40 candidates by straight-line dist
Stage2: filter offline/on-trip -> 26; rank by road ETA + rating + idle

Top-K result:
  1. driver_57  ETA 3.2 min  rating 4.8  (OFFER sent, 15s timer)
  2. driver_12  ETA 4.1 min  rating 4.6
  3. driver_88  ETA 4.9 min  rating 4.9

driver_57 declines -> lock released -> offer driver_12 -> ACCEPT
Output: trip assigned to driver_12, rider pushed ETA 4.1 min
```

**Data:**

```json
// Driver location record kept in the in-memory geo index (latest-only)
{
  "driverId": "driver_57",
  "geohash7": "tdr1bpb",          // ~150m cell; prefix = proximity
  "lat": 12.9611, "lng": 77.6387,
  "online": true, "onTrip": false,
  "vehicle": "sedan", "rating": 4.8,
  "idleSeconds": 92,
  "lastPing": "2026-07-07T09:14:20Z"   // TTL-expire if stale
}
```

**Flow:**

1. yes
2. no/decline
3. latest-only GEOADD
4. Rider requests ride @ lat,lng
5. GEOSEARCH region indexradius 2-5km
6. ~40 candidates
7. Filter: online, free,vehicle match, not locked
8. Rank by ROAD ETA +rating + idle-time fairness
9. Top-K
10. Atomic lock + OFFER to #1WebSocket, 15s TTL
11. Acceptedin time?
12. Assign tripunique active-trip constraint
13. Release lock,offer next candidate
14. Drivers ping GPS 3-5s
15. (Redis GEOper-city shard)

