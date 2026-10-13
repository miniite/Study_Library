---
type: "practice"
difficulty: "Intermediate"
---

# Explain the Observer pattern and give a real use case like a notification system.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Classic LLD Problems](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Classic_LLD_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The interviewer wants me to explain the Observer pattern clearly and then ground it in a real notification system. This is an "intermediate experience" LLD question for backend/fullstack SDE roles, so the bar is: crisp definition, the roles/participants, a correct working example, and practitioner judgment — thread-safety, memory leaks, push-vs-pull, sync-vs-async, and how it maps to real infrastructure (event buses, pub/sub). I should not just recite the GoF definition; I should show I've built this. I'll use an order-status notification scenario (Swiggy/Flipkart flavor): when an order's status changes, multiple downstream consumers (email, SMS, push, analytics, inventory) must react — without the order object knowing who they are.

Clarifiers I'd state: how many observers and how heavy is each; must delivery be synchronous with the state change or can it be async/best-effort; is ordering or delivery-guarantee required; single-process (in-memory Observer) or cross-service (message broker)? These drive whether the classic in-process pattern is enough or we escalate to a queue.

The pattern in one sentence

Observer defines a one-to-many dependency: when one object (the Subject/Observable) changes state, all its registered dependents (Observers) are notified and updated automatically — and the Subject does not know the concrete types of its observers. It decouples "something happened" from "who cares about it."

Participants
Subject/Observable — holds the observer list; exposes subscribe, unsubscribe, and notify. Owns the state.
Observer — an interface with an update(event) method that reacts to change.
ConcreteObserver — the actual reactors (EmailNotifier, SmsNotifier, PushNotifier, AnalyticsLogger).
Client — wires subjects to observers at startup/registration.
First principles: why not just call them directly?

Naively, when order status changes I could write inside the order:

CODE
order.status = DELIVERED;
emailService.send(...);
smsService.send(...);
pushService.send(...);
analytics.log(...);

That hard-wires the order class to every consumer. Adding a WhatsApp notifier means editing the order (OCP violation), and the order now depends on five services (tight coupling, hard to test, hard to reuse). Observer inverts this: the order just announces "my status changed" to a list it doesn't inspect; consumers opt in. New consumer = register a new observer, order untouched.

Real use case: order-status notification system

Flow: OrderService (Subject) changes an order to SHIPPED. It calls notify(orderEvent). Every registered observer's update(event) runs: EmailNotifier sends the shipping email, SmsNotifier texts the tracking link, PushNotifier fires a mobile push, AnalyticsObserver logs a funnel event, InventoryObserver decrements stock. The OrderService has zero knowledge of any of them — it depends only on the OrderObserver interface.

This is exactly the shape of EventEmitter in Node, PropertyChangeListener in Java Swing, RxJS Subscribe, React state → re-render, and Kafka/SNS consumers at the infra scale. Same idea, different transport.

Practitioner concerns (this is what separates a mid-level answer)

1. Synchronous vs asynchronous notify. The naive loop calls each observer inline on the caller's thread. If EmailNotifier blocks on SMTP for 2 seconds, the order transaction is held hostage and one slow observer degrades the whole request. In production I dispatch to an executor/queue: notify enqueues, observers run on worker threads or, better, the Subject publishes to a message broker (SNS/SQS, Kafka, RabbitMQ) and each notifier is a separate consumer. In-process Observer is the pattern; async/broker is how you scale it.

2. Failure isolation. If SmsNotifier throws, it must not stop PushNotifier. Wrap each update in a try/catch, log, and continue — one observer's failure is isolated. With a broker you get retries + dead-letter queues for free.

3. Memory leaks (the classic bug). Observers registered but never unsubscribed keep the Subject holding strong references, so they're never garbage-collected — the "lapsed listener" leak. Always provide unsubscribe, unregister in teardown, or use weak references for long-lived subjects.

4. Push vs pull. Push: Subject passes the full event data to update(event) — simple, no callback. Pull: Subject passes only itself and observers query what they need — flexible but couples the observer back to the Subject's API. I default to push with a small immutable event object.

5. Thread-safety & ordering. The observer list is mutated by subscribe/unsubscribe while notify iterates — use a CopyOnWriteArrayList or snapshot the list before iterating to avoid ConcurrentModificationException. If observers must run in a defined order, make that explicit (priorities); otherwise treat order as undefined.

6. Re-entrancy. An observer that triggers another state change on the same Subject during notify can cause recursion or notify-during-notify bugs — guard against it.

Summary — what a strong answer emphasizes

Observer = one-to-many, publish-on-change, Subject decoupled from concrete observers via an interface. The win is loose coupling + open/closed extension: adding a notifier is a registration, not a code change to the producer. For a notification system it's a natural fit — the order/event source announces state changes and any number of channel notifiers subscribe. The mid-level polish is knowing the operational realities: run observers async so a slow channel doesn't block the request, isolate failures, provide unsubscribe to avoid leaks, snapshot the list for thread safety, and recognize that at scale the in-process Observer becomes a pub/sub broker (SNS/Kafka) — the same pattern, distributed.

Common follow-ups: "Observer vs Pub/Sub?" (Pub/Sub adds a broker/topic between subject and observers, so they're fully decoupled and can live in different processes; Observer is the in-process special case with direct references). "Observer vs Mediator?" (Mediator centralizes many-to-many communication; Observer is one-to-many broadcast). "How do you test it?" (register a mock/spy observer and assert update was called with the right event).

**Comparison:**

```
Concern	Naive direct calls	Observer pattern
Coupling	Producer knows all consumers	Producer knows only the interface
Add a channel	Edit the producer (OCP break)	Register a new observer
Slow consumer	Blocks the whole request	Dispatch async per observer
One consumer fails	Can abort the rest	Isolated via try/catch/DLQ
Testability	Must stub every service	Assert update on a mock observer
Related pattern	How it differs from Observer
Pub/Sub	Adds a broker/topic; subjects & observers fully decoupled, cross-process
Mediator	Centralizes many-to-many coordination, not one-to-many broadcast
Strategy	Selects one interchangeable algorithm; not change-notification
```

**Code:**

```java
// ---- Observer interface: every reactor implements this ----
public interface OrderObserver {
    void update(OrderEvent event);
}

// ---- Immutable event (push model: carry the data) ----
public record OrderEvent(String orderId, OrderStatus status, String customerId) {}

// ---- Subject / Observable ----
public class OrderService {
    // CopyOnWriteArrayList: safe to iterate while subscribe/unsubscribe mutate
    private final List<OrderObserver> observers = new CopyOnWriteArrayList<>();
    private final Executor async = Executors.newFixedThreadPool(4); // don't block caller

    public void subscribe(OrderObserver o)   { observers.add(o); }
    public void unsubscribe(OrderObserver o) { observers.remove(o); } // prevents leaks

    public void updateStatus(String orderId, String customerId, OrderStatus s) {
        // ... persist state change first ...
        OrderEvent e = new OrderEvent(orderId, s, customerId);
        notifyObservers(e);
    }

    private void notifyObservers(OrderEvent e) {
        for (OrderObserver o : observers) {
            async.execute(() -> {                 // async dispatch
                try { o.update(e); }              // failure isolation
                catch (Exception ex) { log.error("observer failed", ex); }
            });
        }
    }
}

// ---- Concrete observers (channels): add new ones without touching OrderService ----
public class EmailNotifier implements OrderObserver {
    public void update(OrderEvent e) { email.send(e.customerId(), "Order " + e.status()); }
}
public class SmsNotifier implements OrderObserver {
    public void update(OrderEvent e) { sms.send(e.customerId(), "Order " + e.status()); }
}
public class PushNotifier implements OrderObserver {
    public void update(OrderEvent e) { push.send(e.customerId(), e.orderId(), e.status()); }
}
public class AnalyticsObserver implements OrderObserver {
    public void update(OrderEvent e) { metrics.increment("order." + e.status()); }
}

// ---- Wiring (client) ----
OrderService svc = new OrderService();
svc.subscribe(new EmailNotifier());
svc.subscribe(new SmsNotifier());
svc.subscribe(new PushNotifier());
svc.subscribe(new AnalyticsObserver());
svc.updateStatus("ORD-42", "cust-7", OrderStatus.SHIPPED); // all 4 react, none coupled to svc
```

**Example input / output:**

```text
subscribe(EmailNotifier)
subscribe(SmsNotifier)
subscribe(PushNotifier)
subscribe(AnalyticsObserver)

updateStatus("ORD-42", "cust-7", SHIPPED)
   -> OrderService persists status = SHIPPED
   -> notifyObservers(OrderEvent{ORD-42, SHIPPED, cust-7})
        EmailNotifier.update     -> email  to cust-7: "Order SHIPPED"
        SmsNotifier.update       -> sms    to cust-7: "Order SHIPPED"
        PushNotifier.update      -> push   to cust-7 for ORD-42
        AnalyticsObserver.update -> metric order.SHIPPED += 1
   (OrderService knew none of the four concrete types)

Later: add WhatsAppNotifier  ->  svc.subscribe(new WhatsAppNotifier())
   OrderService source code: UNCHANGED (Open/Closed).

Failure case: SmsNotifier throws
   -> caught + logged; Email, Push, Analytics still deliver (isolation).
```

**Data:**

```json
// The OrderEvent broadcast to every observer on a status change
{
  "orderId": "ORD-42",
  "status": "SHIPPED",
  "customerId": "cust-7",
  "occurredAt": "2026-07-07T10:15:00Z"
}
```

**Flow:**

1. AnalyticsObserver
2. PushNotifier
3. SmsNotifier
4. EmailNotifier
5. OrderService (Subject)
6. AnalyticsObserver
7. PushNotifier
8. SmsNotifier
9. EmailNotifier
10. OrderService (Subject)
11. par
12. [async dispatch, failure-isolated]
13. Subject holds only List<OrderObserver>,
14. never the concrete channel types
15. updateStatus(ORD-42, SHIPPED) & persist
16. notifyObservers(event)
17. update(event)
18. update(event)
19. update(event)
20. update(event)

