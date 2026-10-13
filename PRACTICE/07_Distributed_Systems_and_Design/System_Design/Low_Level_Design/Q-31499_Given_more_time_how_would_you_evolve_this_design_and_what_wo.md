---
type: "practice"
difficulty: "Intermediate"
---

# Given more time how would you evolve this design and what would you refactor first?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Classic LLD Problems](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Classic_LLD_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

This is a classic end-of-round LLD wrap-up question: "Given more time, how would you evolve this design and what would you refactor first?" It almost always comes after you have designed something concrete on the whiteboard — a parking lot, a rate limiter, a notification service, a splitwise/expense-sharing model, a cache, etc. The interviewer is not looking for a new feature list. They are probing four things:

Self-awareness — do you know the weak spots in your own design, or did you present it as if it were perfect?
Prioritisation / judgment — can you rank refactors by risk and payoff instead of listing everything? "What first?" is the real test.
Evolution vs. rewrite — do you evolve safely (incremental, backward-compatible) or reach for a rewrite?
Engineering maturity — do you talk about extensibility (OCP), testability, coupling, and observability, not just "add more classes"?

Since the exact design isn't given here, the correct approach is to answer this as a reusable framework and then make it concrete on whatever I designed. I'll clarify with the interviewer first, then walk from principles to a prioritised plan.

Clarifying questions I'd ask: "Are we evolving for scale (more traffic), features (more variants), or maintainability (team velocity)? Is backward compatibility with existing callers required? Do we have tests around this code today?" The right refactor order changes completely based on the answer — refactoring for scale first when the real pain is feature-churn is a common mistake.

From first principles — how I decide what to change

Good LLD refactoring is driven by code smells mapped to SOLID/pattern fixes, ordered by a simple rule: refactor first what is (a) most likely to change and (b) riskiest to change today. That intersection is where technical debt actually hurts.

My decision procedure:

Find the change-magnets. Which class do I keep touching every time a requirement changes? In a payment design it's usually the giant if (method == UPI) … else if (CARD) … switch. That is an Open/Closed Principle violation and my #1 refactor.
Find the God class. A class doing parsing + validation + persistence + notification violates Single Responsibility. Splitting it reduces the blast radius of every future change.
Find hidden coupling. Concrete new calls to dependencies (DB, clock, HTTP client) make the class untestable. Introduce Dependency Inversion — depend on interfaces, inject implementations.
Find duplicated conditionals. Repeated type-switches across methods scream Strategy / Polymorphism / State.
Find leaky abstractions. Callers reaching into internals (order.getItems().get(0).getPrice()) — Law of Demeter — mean my public API is wrong.

Only after the design smells are addressed do I evolve for scale (batching, caching, async, sharding), because scaling a badly-factored design just multiplies the mess.

The prioritised plan (what I'd say)

1. First refactor — kill the type-switch / enable extension (OCP). Take the most frequently edited conditional and replace it with a Strategy pattern behind a Factory. Concretely, in an e-commerce checkout (Amazon/Flipkart) the PaymentProcessor with a switch(paymentMethod) becomes a PaymentStrategy interface with UpiStrategy, CardStrategy, etc., selected by a PaymentStrategyFactory. Now adding a wallet is a new class, not an edit to a tested class. This is first because payment methods are the highest-churn axis and touching that switch is the riskiest edit.

2. Break the God object (SRP). Split the class that mixes concerns. E.g. an OrderService that validates, prices, persists, and emails → OrderValidator, PricingEngine, OrderRepository, and an event that a NotificationListener consumes. Smaller units are independently testable and reusable.

3. Invert dependencies for testability (DIP). Replace new MySqlConnection() / new SystemClock() with injected OrderRepository and Clock interfaces. This is low-glamour but unlocks fast unit tests, which is what actually makes future refactors safe. I'd do this early because every later change is safer once the code is testable.

4. Introduce domain events / decouple side-effects. Notifications, analytics, inventory decrement should not be inline in the checkout path. Emit an OrderPlaced event; let subscribers react. This is the Observer pattern and is the seam where async/queues later plug in (SQS/Kafka) without touching core logic — good for Uber/Swiggy-scale later.

5. Then evolve for scale (only now): make writes idempotent (idempotency keys on POST /orders), add caching for read-heavy lookups, batch DB calls, and make the notification path async. These are evolution, not refactor — they add capability.

How I sequence it safely: each step is behind the same public interface, guarded by tests written before the refactor (characterisation tests), shipped incrementally behind a feature flag if it touches runtime behaviour, never a big-bang rewrite. I'd measure with a metric (e.g., cyclomatic complexity of the change-magnet class, or "files touched per feature") to prove the refactor paid off.

What I would explicitly NOT do first
I would not add speculative abstractions for requirements that don't exist ("we might support crypto someday") — YAGNI. Abstraction has a carrying cost.
I would not rewrite from scratch. Strangler-fig / incremental beats rewrite every time in an interview and in real life.
I would not start with performance micro-optimisation before the structure is clean.
Summary — what a strong answer emphasises

A strong answer (1) names concrete smells in your own design, (2) maps each to a SOLID principle or GoF pattern, (3) ranks them by change-frequency × risk and justifies why that one is first (the OCP/Strategy switch usually wins), (4) separates refactor (structure, no behaviour change) from evolve (new capability, scale), and (5) shows a safe rollout: tests first, incremental, backward-compatible, measured. The single most impressive move is honest self-critique — "here's the ugliest part of what I just drew and here's exactly how I'd fix it first."

#### Common mistakes
listing new features instead of refactors; saying "I'd add more design patterns" (pattern-golf) without a smell to justify them; proposing a rewrite; and failing to prioritise ("I'd do all of these") — the question is literally what first.

Likely follow-ups: "Why that one first and not the other?" (change-frequency × risk), "How do you refactor without breaking callers?" (characterisation tests + stable interface + feature flag), and "When would you not refactor?" (code that never changes — leave it alone).

**Comparison:**

```
Refactor (in order)	Smell it fixes	Principle / Pattern	Why this priority
1. Replace payment switch with Strategy+Factory	Type-switch change-magnet	OCP, Strategy, Factory	Highest churn × highest edit risk
2. Split God OrderService	Multiple responsibilities	SRP	Shrinks blast radius of every future change
3. Inject repo/clock/http via interfaces	Hidden coupling, untestable	DIP	Makes all later refactors safe (tests)
4. Emit domain events for side-effects	Inline notifications/analytics	Observer	Seam for async/queues at scale
5. Idempotency, caching, async, batching	Scale pain	— (evolution, not refactor)	Only safe once structure is clean
Aspect	Refactor	Evolve
Goal	Improve structure	Add capability
Behaviour change	No (verified by tests)	Yes
Do it first?	Yes (fix smells)	After structure is clean
Example	Strategy for payments	Async notifications, sharding
```

**Code:**

```java
// BEFORE — the change-magnet: every new payment method edits this tested class (OCP violation)
class PaymentProcessor {
    Receipt pay(String method, Money amount) {
        if (method.equals("UPI"))      { /* upi logic */ }
        else if (method.equals("CARD")){ /* card logic */ }
        else if (method.equals("WALLET")){ /* wallet */ }
        // adding NETBANKING => edit + retest this whole class
        throw new IllegalArgumentException(method);
    }
}

// AFTER — Strategy + Factory: adding a method is a NEW class, not an edit (OCP satisfied)
interface PaymentStrategy { Receipt pay(Money amount); }

class UpiStrategy    implements PaymentStrategy { public Receipt pay(Money a){ /* ... */ return new Receipt(); } }
class CardStrategy   implements PaymentStrategy { public Receipt pay(Money a){ /* ... */ return new Receipt(); } }
class NetbankingStrategy implements PaymentStrategy { public Receipt pay(Money a){ /* new: no edits elsewhere */ return new Receipt(); } }

class PaymentStrategyFactory {
    private final Map<String, PaymentStrategy> registry;      // DIP: injected, testable
    PaymentStrategyFactory(Map<String, PaymentStrategy> registry){ this.registry = registry; }
    PaymentStrategy forMethod(String method){
        PaymentStrategy s = registry.get(method);
        if (s == null) throw new IllegalArgumentException("Unsupported: " + method);
        return s;
    }
}

// SRP + DIP: OrderService orchestrates via interfaces and emits an event instead of doing everything inline
class OrderService {
    private final OrderRepository repo;          // interface, injected (was: new MySqlConn())
    private final PaymentStrategyFactory factory;
    private final EventBus events;               // Observer seam for async later
    OrderService(OrderRepository repo, PaymentStrategyFactory f, EventBus e){ this.repo=repo; this.factory=f; this.events=e; }

    Order place(OrderRequest req) {
        Receipt r = factory.forMethod(req.method()).pay(req.amount());
        Order o = repo.save(new Order(req, r));
        events.publish(new OrderPlaced(o.id()));  // notifications/analytics decoupled, no inline side-effects
        return o;
    }
}
```

**Example input / output:**

```text
Worked example — "add NETBANKING" before vs. after the first refactor:

- BEFORE: developer edits `PaymentProcessor.pay(...)` (a tested, high-traffic method) → risk of regressing UPI/CARD → full re-test of the class. Files touched: 1 shared hot file.
- AFTER (Strategy+Factory): developer adds `NetbankingStrategy` + one registry entry. Existing tested strategies untouched. Files touched: 1 new class + 1 config line. Regression surface: ~0.

Metric to prove it worked: "files touched per new payment method" drops from `PaymentProcessor` + its test to a self-contained new class — and cyclomatic complexity of the core path stays flat as methods grow.
```

**Flow:**

1. Feature churn
2. Scale/traffic
3. Maintainability
4. Given more time
5. What is the pain?
6. Refactor smells first
7. Still: fix smells, then scale
8. 1. Kill type-switch ->Strategy+Factory OCP
9. 2. Split God class -> SRP
10. 3. Inject deps -> DIP, nowtestable
11. 4. Domain events -> Observer,decouple side-effects
12. 5. EVOLVE: idempotency,cache, async, shard
13. Ship incrementally: tests first,feature-flag, measure

