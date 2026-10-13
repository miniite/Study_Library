---
type: "practice"
difficulty: "Deep"
---

# Model the order lifecycle states in a food delivery app. Which pattern captures it?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ State Pattern](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/State_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking. "Model the order lifecycle states in a food delivery app. Which pattern captures it?" has two parts. First, enumerate the real states an order passes through (from creation to a terminal state) and the legal transitions between them — including the unhappy paths (cancellation, payment failure, refund, delivery failure). Second, name and justify the design pattern that best represents a lifecycle: the State pattern (a Gang-of-Four behavioural pattern), and defend why it beats the naive enum + switch approach. At "deep understanding" level the interviewer wants not just "use State pattern" but why, the trade-offs versus a Finite State Machine (FSM) table, how it enforces invariants, how it handles concurrency and persistence, and where it can be over-engineering.

What the interviewer is probing. (1) Do I know the actual domain states (not a toy 3-state list)? (2) Do I understand that an order's behaviour depends on its state, so behaviour should be localized to the state — the essence of the State pattern? (3) Can I compare State-pattern classes vs. a transition-table FSM vs. an enum with a switch? (4) Do I consider guards, side effects (payment capture, refund, notifications), idempotency, concurrency, and audit/event-sourcing? (5) Do I know when NOT to use full State-pattern classes?

#### Assumptions
/ clarifying questions.

One order = one restaurant, immediate (not scheduled) delivery. Cancellation allowed until food is prepared; refunds on cancel-after-payment.
We need an audit trail of every transition (regulatory/CX). I'll therefore emit a domain event on each transition (a nod to event-sourcing).
Transitions are triggered by external actors (customer, restaurant, delivery partner, payment gateway webhook), so guards and idempotency matter.

Plan. List states → define the transition graph with guards/side-effects → present the State pattern as the answer → compare to alternatives → cover concurrency, persistence, idempotency, and pitfalls → summarize.

The states (the actual lifecycle)

A production food-delivery order is not 3 states. A faithful model:

CREATED — cart frozen into an order, not yet paid.
PAYMENT_PENDING — awaiting gateway confirmation (or COD, deferred).
PAYMENT_FAILED — gateway declined; terminal unless retried.
CONFIRMED — paid (or COD accepted) and sent to the restaurant.
ACCEPTED / PREPARING — restaurant accepted and is cooking.
READY_FOR_PICKUP — food ready, awaiting a rider.
OUT_FOR_DELIVERY — rider picked up, en route.
DELIVERED — terminal happy path.
CANCELLED — by customer/restaurant/system; may trigger REFUND_PENDING → REFUNDED.
DELIVERY_FAILED — rider couldn't deliver (customer unreachable); routes to refund/redeliver.

Terminal states: DELIVERED, REFUNDED, PAYMENT_FAILED (final), CANCELLED (if unpaid). Everything else is transient. Each transition has a trigger (who/what causes it), often a guard (a precondition, e.g. "cancellable only before PREPARING"), and a side effect (capture payment, issue refund, notify observers, assign rider).

Which pattern captures it — the State pattern

The defining property of an order is: its allowed behaviour depends on its current state. cancel() means "refund + notify" when CONFIRMED, but "no-op" when already DELIVERED, and "illegal" is impossible because a delivered order shouldn't accept cancel. When behaviour varies by state, the GoF State pattern is the textbook fit: represent each state as its own class implementing a common OrderState interface; the Order (context) holds a reference to the current state object and delegates operations to it. Each state class knows exactly which transitions are legal from it and performs the transition by swapping the context's state.

Why this is the right pattern (deep justification):

Localizes transition logic. Legality lives in the state, not in a giant conditional. Adding RETURN_INITIATED is a new class + wiring, touching nothing else (Open/Closed).
Makes illegal transitions structurally impossible. DeliveredState simply doesn't implement cancel() meaningfully → calling it throws or no-ops. You can't "deliver an unpaid order" because PaymentPendingState has no markDelivered.
Eliminates switch/if sprawl. The naive design scatters if (status == CONFIRMED) across services; every new state edits every switch — bug-prone and untestable. State pattern gives one class per state, unit-testable in isolation.
Encapsulates side effects with the transition (payment capture on confirm, refund on cancel), keeping them consistent.

The State pattern is one concrete implementation of a Finite State Machine (FSM). The two viable implementations are (a) State-pattern classes, and (b) a transition table (Map<(State,Event), State> + guard/action functions). For a domain with many states and configurable rules, a table-driven FSM (or a workflow engine like Temporal/Camunda / AWS Step Functions) is often better because the graph is data, visible at a glance, and editable without recompiling. For a modest, code-owned lifecycle, State-pattern classes read best and keep behaviour close to state. A strong answer names both and picks based on how dynamic the graph is.

Concurrency, persistence, idempotency (the deep part)

Real orders are mutated by concurrent, out-of-order external events (a gateway webhook, a rider app, a customer tap). So beyond the pattern:

Guarded transitions + idempotency: the same webhook can arrive twice. confirm() from CONFIRMED must be a no-op, not a double-capture. Each transition checks "am I already there?" and dedupes on an event id.
Optimistic concurrency: persist the order with a version; a transition does a compare-and-swap (UPDATE … WHERE id=? AND version=?). If it loses the race, reload and re-evaluate — the FSM stays consistent because the loser sees the winner's state and its guard rejects the now-illegal move.
Persistence of state: store the current state as a string/enum column, but also append a transition log / event stream (OrderStateChanged{from,to,trigger,at}). This gives an audit trail and lets you rebuild state (event sourcing) — invaluable for disputes ("when was it marked delivered?").
Timeouts: a PAYMENT_PENDING order that never confirms must auto-cancel; a scheduler drives a time-based transition. FSMs model this as a timer event.
When it's over-engineering

For a truly trivial lifecycle (2–3 states, no branching), a single enum column and a validated setStatus may be enough — full State-pattern classes add ceremony. And if the graph is business-configurable or spans services with retries/compensation, jump straight to a workflow/orchestration engine rather than hand-rolling — the pattern doesn't scale to long-running distributed sagas.

Summary — what a strong answer emphasizes

Enumerate the real states including unhappy paths; identify the lifecycle as a Finite State Machine and pick the State pattern (behaviour varies by state → put behaviour in the state) as the class-level realization; contrast it with enum-switch (rejected: sprawl) and table-driven FSM / workflow engines (preferred when the graph is dynamic or distributed); and defend it with invariants-by-construction, guards, idempotency, optimistic concurrency, and an append-only transition log for audit. Naming the pattern is table stakes; the trade-offs and correctness reasoning are what separate a senior answer.

Common follow-up probes
"Add a RETURN/REORDER state" → show it's one class + edges.
"Two webhooks arrive at once" → optimistic locking + idempotent transitions.
"How do you test this?" → each state class tested in isolation; a property test that no illegal edge exists in the transition map.
"Enum vs State classes vs Temporal?" → decision by graph size, dynamism, and distribution.

**Comparison:**

```
From state	Trigger	To state	Guard / side effect
PAYMENT_PENDING	CONFIRM	CONFIRMED	capture payment
PAYMENT_PENDING	FAIL_PAYMENT	PAYMENT_FAILED	terminal
PAYMENT_PENDING	CANCEL	CANCELLED	unpaid → no refund
CONFIRMED	ACCEPT	PREPARING	restaurant accepts
CONFIRMED	CANCEL	REFUND_PENDING	issue refund
PREPARING	READY	READY_FOR_PICKUP	cancellation now blocked (guard)
READY_FOR_PICKUP	PICK_UP	OUT_FOR_DELIVERY	assign/attach rider
OUT_FOR_DELIVERY	DELIVER	DELIVERED	terminal (happy)
OUT_FOR_DELIVERY	FAIL_DELIVERY	REFUND_PENDING	issue refund
REFUND_PENDING	(auto)	REFUNDED	terminal
Approach	Pros	Cons	Use when
enum + switch	Simplest to start	Logic sprawls across services; every new state edits all switches; hard to test	2–3 static states, one place
State pattern (classes)	Behaviour localized in state; illegal moves impossible; OCP; unit-testable per state	More classes; graph not visible in one glance	Code-owned, moderate lifecycle (this answer)
Table-driven FSM	Graph is data, editable, visualizable	Actions/guards live elsewhere; less OO cohesion	Many states / configurable rules
Workflow engine (Temporal/Step Fn)	Durable, retries, timers, distributed sagas, built-in audit	Infra + learning cost	Long-running, cross-service, compensation
```

**Code:**

```

// ===== State pattern realization of the order FSM ===== public enum Trigger { CONFIRM, ACCEPT, READY, PICK_UP, DELIVER, CANCEL, FAIL_PAYMENT, FAIL_DELIVERY }

public interface OrderState { String name(); // default: every trigger is illegal unless a concrete state overrides it default OrderState on(Trigger t, Order ctx) { throw new IllegalStateTransition(name() + " cannot handle " + t); } default boolean isTerminal() { return false; } }

public class PaymentPendingState implements OrderState { public String name() { return "PAYMENT_PENDING"; } public OrderState on(Trigger t, Order o) { return switch (t) { case CONFIRM -> { o.capturePayment(); yield new ConfirmedState(); } case FAIL_PAYMENT -> new PaymentFailedState(); case CANCEL -> new CancelledState(); // unpaid -> no refund needed default -> OrderState.super.on(t, o); }; } } public class ConfirmedState implements OrderState { public String name() { return "CONFIRMED"; } public OrderState on(Trigger t, Order o) { return switch (t) { case ACCEPT -> new PreparingState(); case CANCEL -> { o.issueRefund(); yield new RefundPendingState(); } // paid -> refund default -> OrderState.super.on(t, o); }; } } public class PreparingState implements OrderState { public String name() { return "PREPARING"; } public OrderState on(Trigger t, Order o) { // guard: no cancellation once cooking has started return t == Trigger.READY ? new ReadyForPickupState() : OrderState.super.on(t, o); } } public class ReadyForPickupState implements OrderState { public String name() { return "READY_FOR_PICKUP"; } public OrderState on(Trigger t, Order o) { return t == Trigger.PICK_UP ? new OutForDeliveryState() : OrderState.super.on(t, o); } } public class OutForDeliveryState implements OrderState { public String name() { return "OUT_FOR_DELIVERY"; } public OrderState on(Trigger t, Order o) { return switch (t) { case DELIVER -> new DeliveredState(); case FAIL_DELIVERY -> { o.issueRefund(); yield new RefundPendingState(); } default -> OrderState.super.on(t, o); }; } } public class DeliveredState implements OrderState { public String name(){return "DELIVERED";} public boolean isTerminal(){return true;} } public class PaymentFailedState implements OrderState { public String name(){return "PAYMENT_FAILED";}public boolean isTerminal(){return true;} } public class CancelledState implements OrderState { public String name(){return "CANCELLED";} public boolean isTerminal(){return true;} } public class RefundPendingState implements OrderState { public String name(){ return "REFUND_PENDING"; } public OrderState on(Trigger t, Order o){ return t==Trigger.DELIVER? this : new RefundedState(); } } public class RefundedState implements OrderState { public String name(){return "REFUNDED";} public boolean isTerminal(){return true;} }

// ===== Context (the Order) with idempotency + optimistic concurrency ===== public class Order { final String id; long version; // optimistic lock private OrderState state = new PaymentPendingState(); final List<StateChange> log = new ArrayList<>(); // append-only audit / event stream private final Set<String> seenEventIds = new HashSet<>();

CODE
Copy
public synchronized void fire(Trigger t, String eventId) {
    if (!seenEventIds.add(eventId)) return;          // idempotent: dedupe retried events
    OrderState from = state;
    OrderState to   = state.on(t, this);             // throws if illegal
    this.state = to;
    this.version++;
    log.add(new StateChange(from.name(), to.name(), t, Instant.now()));
    publish(new OrderStateChanged(id, from.name(), to.name(), t));
}
public String status(){ return state.name(); }
void capturePayment(){ /* gateway capture */ }
void issueRefund(){ /* gateway refund */ }
void publish(Object evt){ /* to Kafka / outbox */ }
record StateChange(String from, String to, Trigger t, Instant at) {}
record OrderStateChanged(String id, String from, String to, Trigger t) {}

} class IllegalStateTransition extends RuntimeException { IllegalStateTransition(String m){ super(m);} }

CODE
Copy

```java
// ===== Alternative: table-driven FSM (better when the graph is dynamic/configurable) =====
record Edge(String from, Trigger on) {}
Map<Edge,String> TRANSITIONS = Map.of(
    new Edge("PAYMENT_PENDING", Trigger.CONFIRM), "CONFIRMED",
    new Edge("CONFIRMED",       Trigger.ACCEPT),  "PREPARING",
    new Edge("PREPARING",       Trigger.READY),   "READY_FOR_PICKUP"
    /* ... */);
// transition = lookup; missing key => illegal. Graph is DATA, not code.
```

**Example input / output:**

```text
HAPPY PATH (fire triggers in order):
  start        -> PAYMENT_PENDING
  CONFIRM      -> capturePayment(); PAYMENT_PENDING -> CONFIRMED
  ACCEPT       -> CONFIRMED        -> PREPARING
  READY        -> PREPARING        -> READY_FOR_PICKUP
  PICK_UP      -> READY_FOR_PICKUP -> OUT_FOR_DELIVERY
  DELIVER      -> OUT_FOR_DELIVERY -> DELIVERED   (terminal)

CANCEL AFTER PAYMENT:
  ... CONFIRMED
  CANCEL       -> issueRefund(); CONFIRMED -> REFUND_PENDING -> REFUNDED

IDEMPOTENCY (duplicate webhook):
  fire(CONFIRM, "evt_pay_ok")  -> CONFIRMED
  fire(CONFIRM, "evt_pay_ok")  -> no-op (eventId already seen; no double capture)

ILLEGAL TRANSITION (guard):
  state = PREPARING
  fire(CANCEL, "evt_x")  -> IllegalStateTransition("PREPARING cannot handle CANCEL")

  state = PAYMENT_PENDING
  fire(DELIVER, "evt_y") -> IllegalStateTransition("PAYMENT_PENDING cannot handle DELIVER")
```

**Data:**

```json
// Append-only transition log persisted alongside the order (audit + event sourcing)
{
  "orderId": "ord_5f3a",
  "currentState": "OUT_FOR_DELIVERY",
  "version": 4,
  "log": [
    { "from": "PAYMENT_PENDING", "to": "CONFIRMED",        "trigger": "CONFIRM",  "at": "2026-07-02T12:04:11Z", "eventId": "evt_pay_ok" },
    { "from": "CONFIRMED",        "to": "PREPARING",        "trigger": "ACCEPT",   "at": "2026-07-02T12:04:40Z", "eventId": "evt_rest_ok" },
    { "from": "PREPARING",        "to": "READY_FOR_PICKUP", "trigger": "READY",    "at": "2026-07-02T12:18:02Z", "eventId": "evt_ready" },
    { "from": "READY_FOR_PICKUP", "to": "OUT_FOR_DELIVERY", "trigger": "PICK_UP",  "at": "2026-07-02T12:22:31Z", "eventId": "evt_pickup" }
  ]
}
```

**Flow:**

1. CONFIRM / capture
2. FAIL_PAYMENT
3. CANCEL
4. ACCEPT
5. CANCEL / refund
6. READY
7. PICK_UP
8. DELIVER
9. FAIL_DELIVERY / refund
10. PAYMENT_PENDING
11. CONFIRMED
12. PAYMENT_FAILED
13. CANCELLED
14. PREPARING
15. REFUND_PENDING
16. READY_FOR_PICKUP
17. OUT_FOR_DELIVERY
18. DELIVERED
19. REFUNDED

