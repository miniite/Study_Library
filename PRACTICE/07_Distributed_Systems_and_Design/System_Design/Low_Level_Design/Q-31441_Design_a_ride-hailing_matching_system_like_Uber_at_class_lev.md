---
type: "practice"
difficulty: "Deep"
---

# Design a ride-hailing matching system like Uber at class level.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Classic LLD Problems](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Classic_LLD_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking. "Design a ride-hailing matching system like Uber at class level" is an LLD prompt focused on the matching subsystem: given a rider's trip request and a pool of nearby available drivers, model the classes that (1) index driver locations, (2) find candidates, (3) rank/select the best driver, (4) offer the trip and handle accept/reject/timeout, and (5) drive the trip through its lifecycle. "At class level" means classes, interfaces, responsibilities, and method signatures with the right design patterns — not the full HLD (Kafka topics, cell sharding, S2/H3 index infra, region failover). But at "deep understanding" level I must show awareness of the matching algorithm, the geo-index abstraction, concurrency (double-booking a driver is the cardinal sin), pluggable strategies (nearest vs. ETA vs. batched dispatch), and surge.

What the interviewer is probing. (1) Can I isolate the matching problem from everything else? (2) Do I abstract the geospatial index behind an interface so nearest-driver lookup is O(log n)/bounded, not a full scan? (3) Do I make the matching policy pluggable (Strategy) — greedy-nearest vs. lowest-ETA vs. batch/global-optimal? (4) Do I handle the concurrency invariant: a driver can be offered/assigned to exactly one trip at a time — with locking/atomic reservation and offer timeouts? (5) Do I model the offer/accept/reject/timeout dispatch protocol and the trip lifecycle (State pattern)? (6) Surge/pricing as another Strategy, and Observer for driver-location updates and rider notifications.

#### Assumptions
/ clarifying questions.

Single trip type (UberGo) to start; design so pool/auto/premium are additive.
Matching = 1 rider ↔ 1 driver; batched matching is an extension.
Driver locations stream in (~every 4s). We index them in an in-memory geo structure fronted by an interface (real impl: Redis GEO / S2 / H3 / QuadTree).
Offer timeout ~15s; on reject/timeout, offer the next candidate. Reserve a driver before offering so no two riders get the same driver.

Plan. Actors → entities → geo-index abstraction → candidate search → matching Strategy → offer/dispatch protocol with concurrency control → trip State machine → surge Strategy + Observer → wire the flow → trade-offs, failure modes, pitfalls.

From first principles to the design
1. Entities and value objects
Rider / Driver — driver has Location, a DriverStatus (OFFLINE / AVAILABLE / OFFERED / ON_TRIP), a vehicle, rating.
Value objects (immutable): Location(lat, lng), GeoHash/CellId, Fare, Eta.
TripRequest — riderId, pickup Location, drop, requested vehicle type, timestamp.
Trip — id, rider, matched driver, fare, and a TripState (State pattern) covering the trip lifecycle.
Offer — a time-boxed proposal of a trip to a driver, with an expiry.
2. The geospatial index is an interface (the key abstraction)

Finding candidates cannot be a linear scan of millions of drivers. I hide the spatial structure behind DriverLocationIndex with nearby(Location, radiusMeters) -> List<DriverId> and updateLocation(driverId, Location). Concrete implementations: a QuadTree/KD-tree in-memory, or geohash/S2/H3 cell buckets, or Redis GEO (GEOSEARCH). Programming to the interface means I can swap the index without touching matching logic, and I get bounded lookups (query only the pickup cell + its 8 neighbours). This abstraction is the single most important LLD decision here.

3. Candidate search + Matching is a Strategy

MatchingStrategy.selectDriver(TripRequest, List<Candidate>) -> Optional<DriverId> lets me swap policies without changing the orchestrator:

NearestAvailableStrategy — greedy by haversine distance (simple, fast).
LowestEtaStrategy — rank by routed ETA (uses live traffic; better UX than raw distance).
BatchDispatchStrategy — accumulate requests for a short window and solve a global assignment (Hungarian/min-cost matching) to minimize total wait across many riders — better fleet efficiency, higher latency.

This is the deep insight: greedy-nearest is locally optimal but globally wasteful; senior systems batch and optimize. Naming the trade-off (latency vs. global efficiency) earns the "deep understanding" credit.

4. Concurrency: reserve-then-offer (the cardinal invariant)

The nastiest bug in ride-matching is double-booking: two riders both offered the same driver. I enforce "a driver can be in flight for at most one offer" with an atomic reservation: before sending an offer, DriverReservation.tryReserve(driverId) does a compare-and-set of AVAILABLE → OFFERED (a ConcurrentHashMap.replace / Redis SET NX / DB UPDATE … WHERE status='AVAILABLE'). Only the winner offers. On accept → OFFERED → ON_TRIP; on reject/timeout → release back to AVAILABLE and offer the next candidate. This makes the invariant safe under concurrency without a global lock.

5. Offer / dispatch protocol

DispatchService implements the offer loop:

index.nearby(pickup, r) → candidate driver ids.
MatchingStrategy.selectDriver(...) → ranked pick.
reservation.tryReserve(driver); if lost, try next candidate.
Send Offer (with a timer); await accept/reject/timeout.
Accept → create Trip, driver ON_TRIP, notify rider (Observer). Reject/Timeout → release, next candidate. Exhausted → widen radius or fail with "no drivers".

The offer timeout is essential: a non-responding driver must not freeze a rider. A ScheduledExecutor/timer wheel expires offers.

6. Trip lifecycle is a State machine

Once matched, the trip runs through REQUESTED → DRIVER_ASSIGNED → DRIVER_ARRIVED → IN_PROGRESS → COMPLETED, with CANCELLED branches. Same reasoning as any lifecycle: use the State pattern so legal transitions and side effects (start fare meter, capture payment) live in the state, and illegal moves are impossible.

7. Surge pricing is a Strategy; location updates use Observer

PricingStrategy.quote(TripRequest, DemandSnapshot) -> Fare computes base + surge from supply/demand in the pickup cell — swappable and independent of matching. Driver apps stream location updates; the index is the Observer target so freshness is decoupled from consumers (matching, ETA, rider live-tracking).

The end-to-end match flow

RideMatchingService.requestRide(tripRequest):

pricing.quote(...) → show fare/surge to rider.
dispatch.match(tripRequest) runs the offer loop (search → strategy → reserve → offer → accept).
On accept, create Trip (State DRIVER_ASSIGNED), attach driver, notify rider.
Trip advances via State pattern to completion; on completion, driver released to AVAILABLE, payment captured.

#### Trade-offs
, failure modes, pitfalls (deep understanding)
Greedy vs. batch: greedy-nearest minimizes this rider's ETA; batch minimizes fleet wait — pick per business goal; often greedy at low load, batch at surge.
Double-booking is the classic failure — always atomic reserve, never "read status then offer" (TOCTOU race).
Offer starvation: always time-box offers; widen radius on exhaustion; cap total attempts and fail gracefully.
Stale locations: an index entry older than N seconds is untrustworthy → TTL/evict; ranking should penalize stale positions.
Thundering herd on one nearby driver: reservation CAS ensures only one wins; others move to the next candidate.
Distance ≠ ETA: haversine is a proxy; use routed ETA where possible (a one-way river/flyover makes the "nearest" driver 12 min away).
Over-engineering: don't hand-roll a distributed global-optimal solver up front; start greedy-nearest behind the Strategy interface and evolve.
Summary — what a strong answer emphasizes

Isolate matching; put the geo-index behind an interface for bounded nearest-driver search; make the matching policy a Strategy (nearest / lowest-ETA / batch, with the latency-vs-efficiency trade-off stated); enforce the one-driver-one-offer invariant with atomic reserve-then-offer plus offer timeouts; run the trip through a State machine; keep surge pricing as a Strategy and location updates as Observer. The class model must be substitutable (SOLID), and the answer must foreground concurrency (double-booking) and the greedy-vs-global trade-off — that's what elevates it beyond "just find the nearest driver."

Common follow-up probes
"Two requests target the same driver at once" → reserve CAS, loser retries next candidate.
"Nearest driver is 10 min by road" → switch Strategy to lowest routed ETA.
"Improve fleet efficiency at surge" → batch dispatch + min-cost matching.
"Which geo-index?" → QuadTree/S2/H3/Redis GEO; query cell + neighbours; TTL stale entries.

**Comparison:**

```
Concern	Class / Interface	Pattern	Why (what varies)
Nearest-driver lookup	DriverLocationIndex	Abstraction / Strategy	Swap QuadTree/S2/H3/Redis GEO; bounded search, not full scan
Which driver to pick	MatchingStrategy	Strategy	nearest vs. lowest-ETA vs. batch; policy changes with load/goal
One-offer-per-driver	DriverReservation	Atomic CAS / lock	Prevents double-booking (TOCTOU race)
Offer/accept/reject/timeout	DispatchService, Offer	Protocol + timer	Drivers may not respond; must not freeze rider
Trip lifecycle	TripState + *State	State	Legal transitions & side-effects localized
Surge pricing	PricingStrategy	Strategy	Pricing rules independent of matching
Live location updates	DriverLocationIndex.updateLocation	Observer	Decouple driver pings from consumers
Use-case wiring	RideMatchingService	Facade	Thin orchestrator over the flow
Matching policy	Optimizes	Cost	When
Nearest (haversine)	This rider's raw distance	Fast, simple; ignores traffic	Low load, prototype
Lowest routed ETA	This rider's real wait	Needs routing/ETA service	Default UX
Batch / min-cost assignment	Whole-fleet total wait	Higher latency, complex	Surge / dense supply
Failure mode	Cause	Mitigation in design
Double-booking	Read-status-then-offer race	Atomic tryReserve CAS AVAILABLE→OFFERED
Rider stuck	Driver ignores offer	15s offer timeout → release → next candidate
"Nearest" is far by road	Distance ≠ ETA	LowestEtaStrategy (routed ETA)
Ghost driver	Stale location	TTL/evict stale entries; penalize old pings
```

**Code:**

```java
// ===== Value objects =====
public record Location(double lat, double lng) {
    public double distanceMeters(Location o) { /* haversine */ return Haversine.dist(this, o); }
}
public record Fare(long paise) {}
public record Eta(int seconds) {}

// ===== Driver / status =====
public enum DriverStatus { OFFLINE, AVAILABLE, OFFERED, ON_TRIP }
public class Driver {
    final String id; volatile Location location; volatile DriverStatus status = DriverStatus.OFFLINE;
    double rating; Instant lastSeen;
}

// ===== Geo-index abstraction (swap QuadTree / S2 / H3 / Redis GEO) =====
public interface DriverLocationIndex {
    void updateLocation(String driverId, Location loc);   // Observer target for driver pings
    List<String> nearby(Location center, int radiusMeters);
}
public class QuadTreeIndex implements DriverLocationIndex {
    public void updateLocation(String id, Location l) { /* reinsert into quadtree */ }
    public List<String> nearby(Location c, int r) { /* query cell + neighbours */ return List.of(); }
}

// ===== Matching Strategy =====
public record Candidate(String driverId, Location loc, double rating, Eta eta) {}
public interface MatchingStrategy { Optional<String> selectDriver(TripRequest req, List<Candidate> cands); }

public class NearestAvailableStrategy implements MatchingStrategy {
    public Optional<String> selectDriver(TripRequest req, List<Candidate> c) {
        return c.stream().min(Comparator.comparingDouble(x -> x.loc().distanceMeters(req.pickup())))
                .map(Candidate::driverId);
    }
}
public class LowestEtaStrategy implements MatchingStrategy {
    public Optional<String> selectDriver(TripRequest req, List<Candidate> c) {
        return c.stream().min(Comparator.comparingInt(x -> x.eta().seconds())).map(Candidate::driverId);
    }
}

// ===== Atomic reservation: enforces one-offer-per-driver (no double-booking) =====
public class DriverReservation {
    private final Map<String, Driver> drivers;   // shared registry
    DriverReservation(Map<String, Driver> d) { this.drivers = d; }
    /** CAS AVAILABLE -> OFFERED; only the winner may offer. */
    public boolean tryReserve(String driverId) {
        Driver d = drivers.get(driverId);
        synchronized (d) {                        // or Redis SET NX / DB UPDATE ... WHERE status='AVAILABLE'
            if (d.status == DriverStatus.AVAILABLE) { d.status = DriverStatus.OFFERED; return true; }
            return false;
        }
    }
    public void release(String driverId) {
        Driver d = drivers.get(driverId);
        synchronized (d) { if (d.status == DriverStatus.OFFERED) d.status = DriverStatus.AVAILABLE; }
    }
    public void confirm(String driverId) {
        Driver d = drivers.get(driverId);
        synchronized (d) { d.status = DriverStatus.ON_TRIP; }
    }
}

// ===== Offer / dispatch loop =====
public class Offer { String tripReqId, driverId; Instant expiresAt; }
public enum OfferResult { ACCEPTED, REJECTED, TIMEOUT }

public class DispatchService {
    private final DriverLocationIndex index;
    private final MatchingStrategy strategy;
    private final DriverReservation reservation;
    private final int OFFER_TIMEOUT_S = 15, MAX_ATTEMPTS = 8;

    public Optional<String> match(TripRequest req) {
        int radius = 1500;
        Set<String> tried = new HashSet<>();
        for (int attempt = 0; attempt < MAX_ATTEMPTS; attempt++) {
            List<Candidate> cands = buildCandidates(index.nearby(req.pickup(), radius), req, tried);
            Optional<String> pick = strategy.selectDriver(req, cands);
            if (pick.isEmpty()) { radius *= 2; continue; }            // widen search
            String driver = pick.get(); tried.add(driver);
            if (!reservation.tryReserve(driver)) continue;            // lost the race -> next candidate
            OfferResult r = sendOfferAndAwait(new Offer(), OFFER_TIMEOUT_S);
            if (r == OfferResult.ACCEPTED) { reservation.confirm(driver); return Optional.of(driver); }
            reservation.release(driver);                              // reject/timeout -> free + retry
        }
        return Optional.empty();                                     // no driver found
    }
    private List<Candidate> buildCandidates(List<String> ids, TripRequest r, Set<String> tried){ /*...*/ return List.of(); }
    private OfferResult sendOfferAndAwait(Offer o, int timeoutS){ /* push to driver app, await/timeout */ return OfferResult.ACCEPTED; }
}

// ===== Trip lifecycle (State pattern) =====
public interface TripState { String name(); default TripState next(){ return this; } }
public class DriverAssignedState implements TripState { public String name(){return "DRIVER_ASSIGNED";} public TripState next(){return new DriverArrivedState();} }
public class DriverArrivedState  implements TripState { public String name(){return "DRIVER_ARRIVED";}  public TripState next(){return new InProgressState();}    }
public class InProgressState     implements TripState { public String name(){return "IN_PROGRESS";}     public TripState next(){return new CompletedState();}     }
public class CompletedState      implements TripState { public String name(){return "COMPLETED";} }

// ===== Surge pricing Strategy =====
public interface PricingStrategy { Fare quote(TripRequest req, double demandSupplyRatio); }
public class SurgePricing implements PricingStrategy {
    public Fare quote(TripRequest req, double ratio) {
        double mult = Math.max(1.0, Math.min(3.0, ratio));    // clamp 1.0x .. 3.0x
        long base = 5000 + (long)(req.estimatedKm() * 1200);  // ₹50 base + ₹12/km
        return new Fare((long)(base * mult));
    }
}

// ===== Orchestrator (Facade) =====
public class RideMatchingService {
    private final DispatchService dispatch; private final PricingStrategy pricing;
    public Trip requestRide(TripRequest req, double demandRatio) {
        Fare quote = pricing.quote(req, demandRatio);         // show to rider
        String driver = dispatch.match(req).orElseThrow(() -> new NoDriversAvailable());
        Trip trip = new Trip(req.riderId(), driver, quote, new DriverAssignedState());
        // notify rider (Observer) ...
        return trip;
    }
}
```

**Example input / output:**

```text
INPUT:
  rider pickup = (12.9716, 77.5946), vehicleType = UberGo
  index.nearby(pickup, 1500m) -> [drv_17, drv_09, drv_31]
  candidates (routed ETA):  drv_17=4min, drv_09=6min, drv_31=3min(stale 40s)

DRY RUN (LowestEtaStrategy + reserve-then-offer):
  strategy.selectDriver -> drv_31 (3 min)      // lowest ETA
  reservation.tryReserve(drv_31) -> FALSE       // already OFFERED to another rider (race)
  next -> drv_17 (4 min)
  reservation.tryReserve(drv_17) -> TRUE (AVAILABLE -> OFFERED)
  sendOfferAndAwait(drv_17, 15s) -> ACCEPTED
  reservation.confirm(drv_17)   (OFFERED -> ON_TRIP)
  create Trip{ id=trp_881, driver=drv_17, state=DRIVER_ASSIGNED, fare=₹216, surge=1.2x }

OUTPUT:
  Trip{ trp_881, driver=drv_17, DRIVER_ASSIGNED, ₹216 }

TIMEOUT PATH:
  offer drv_17 -> no response in 15s -> TIMEOUT
  reservation.release(drv_17) (OFFERED -> AVAILABLE)
  offer next candidate ...  (if MAX_ATTEMPTS exhausted -> NoDriversAvailable, widen radius first)

NO DRIVERS:
  index.nearby returns [] even at max radius -> throw NoDriversAvailable
```

**Data:**

```json
// TripRequest payload and a matched-trip result
{
  "tripRequest": {
    "riderId": "usr_42",
    "pickup": { "lat": 12.9716, "lng": 77.5946 },
    "drop":   { "lat": 12.9352, "lng": 77.6245 },
    "vehicleType": "UberGo",
    "requestedAt": "2026-07-02T09:15:00Z"
  },
  "matchResult": {
    "tripId": "trp_881",
    "driverId": "drv_17",
    "state": "DRIVER_ASSIGNED",
    "farePaise": 21600,
    "surgeMultiplier": 1.2,
    "driverEtaSeconds": 240
  }
}
```

**Flow:**

1. none
2. pick driver
3. lost race
4. reserved
5. ACCEPTED
6. REJECT/TIMEOUT
7. empty at max radius
8. requestRide TripRequest
9. PricingStrategy.quote surgefare
10. index.nearby pickup, radius
11. MatchingStrategy.selectDriver
12. widen radius / attempts++
13. reservation.tryReserve CAS
14. send Offer + 15s timer
15. confirm ON_TRIP -> create TripDRIVER_ASSIGNED
16. release AVAILABLE
17. NoDriversAvailable
18. [Trip State machine:ASSIGNED->ARRIVED->IN_PROGRESS->COMPLETED]
19. driver reaches pickup
20. rider on board / start meter
21. drop-off / capture fare
22. rider/driver cancels
23. DRIVER_ASSIGNED
24. DRIVER_ARRIVED
25. IN_PROGRESS
26. COMPLETED
27. CANCELLED

