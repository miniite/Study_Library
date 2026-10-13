---
type: "practice"
difficulty: "Intermediate"
---

# What is coupling and cohesion? How do you aim for low coupling and high cohesion?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design Pattern Governance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked. Coupling and cohesion are the two oldest and most important metrics of module quality. This is an intermediate/practitioner question, so the interviewer wants more than textbook definitions — they want to see that I can recognise bad coupling/cohesion in real code, name the different kinds, and describe the concrete techniques I use (interfaces, DI, single responsibility, events) to push toward the ideal: low coupling, high cohesion. They may probe with "give an example from your own code" or "how did you refactor it."

Two-sentence definitions to anchor. Cohesion measures how strongly the responsibilities inside one module belong together — high cohesion means the module does one well-defined thing. Coupling measures how much one module depends on the internals of other modules — low coupling means modules interact through small, stable contracts and can change independently.

Plan. Define both with a memorable mental model → describe the spectrum (types of each) so I show depth → give a concrete bad-vs-good refactor → list the practical techniques I use → note trade-offs and how the two interact → summary + common mistakes.

The mental model

Think of a codebase as parts in a machine. Cohesion is how focused each part is (a wheel does "wheel things," not braking + steering + music). Coupling is how tangled the parts are (does swapping the stereo force you to rebuild the engine?). Good design maximises the first and minimises the second. They pull in the same direction: when you split a low-cohesion "god class" into focused pieces, you naturally define clean seams between them, which lowers coupling.

Cohesion — the spectrum (best to worst)
Functional (best): every element contributes to one single task. PasswordHasher only hashes/verifies passwords. This is the target.
Sequential / communicational: elements share data and form a pipeline (parse → validate → transform). Still good.
Temporal: grouped only because they run at the same time (an init() that opens a DB, loads config, and warms a cache). Weak — those don't really belong together.
Logical: grouped because they're "similar kind" but selected by a flag (doThing(type) with a giant switch). Weak.
Coincidental (worst): a Utils/Helpers grab-bag of unrelated functions. Avoid.

A cohesive class has a name you can state its job from in one sentence, few reasons to change (Single Responsibility Principle), and methods that mostly touch the same fields.

Coupling — the spectrum (loose to tight)
Data coupling (best): modules pass only simple data/parameters they need. sendEmail(to, subject, body).
Stamp coupling: you pass a whole record when the callee needs one field (passing the entire Order to get order.total). Mild smell — pass the field.
Control coupling: one module passes a flag that dictates the other's internal control flow (render(mode) with if mode == ...). The caller must know the callee's internals.
Common/global coupling: modules share mutable global state — a change anywhere ripples everywhere.
Content coupling (worst): one module reaches into another's internals/private fields. Any refactor of the internals breaks callers.

Practical signals of high coupling: a small change forces edits across many files ("shotgun surgery"); you can't unit-test class A without spinning up B, C, D; a compile-time dependency cycle between packages.

Concrete refactor

Bad (low cohesion + high coupling): an OrderService that validates the cart, talks to the SQL driver directly, computes tax, formats an HTML email, and calls Stripe's SDK inline. It has five reasons to change, cannot be tested without a real database and network, and every change to email formatting risks breaking checkout.

Good: split by responsibility and depend on interfaces, not concretions:

OrderValidator — validation only (functional cohesion).
OrderRepository (interface) — persistence; PostgresOrderRepository implements it.
TaxCalculator — tax rules only.
PaymentGateway (interface) — StripeGateway implements it.
Notifier (interface) — email/SMS. OrderService now orchestrates these injected collaborators. Each piece is highly cohesive; OrderService depends only on small stable interfaces (data coupling), so I can unit-test it with mocks and swap Postgres→Dynamo or Stripe→Razorpay without touching the others.
How I aim for low coupling + high cohesion (practical techniques)
Single Responsibility Principle — one reason to change per module. If I can't name the class's job in one phrase without "and," I split it. This drives cohesion up.
Depend on abstractions (interfaces), inject dependencies — the Dependency Inversion Principle. OrderService takes a PaymentGateway, not StripeGateway. This is the single biggest lever for low coupling and testability.
Program to the smallest interface (Interface Segregation) — narrow contracts so callers depend on only what they use; reduces stamp/control coupling.
Encapsulate — no reaching into internals. Keep fields private, expose behaviour, obey the Law of Demeter ("don't talk to strangers": a.getB().getC().do() couples you to three classes). This kills content coupling.
Prefer data coupling over control coupling — return polymorphic objects or use Strategy instead of passing behaviour-selecting flags.
Decouple with events/queues at the seams — publish OrderPlaced and let a Notifier subscribe, instead of OrderService calling the emailer directly. The producer no longer knows its consumers (temporal + logical decoupling), which is how services stay independent at scale.
Watch package dependencies — no cycles; dependencies should point toward stable, abstract modules.

#### Trade-offs
and how the two relate
The two metrics reinforce each other: raising cohesion (splitting responsibilities) usually lowers coupling because you create clean seams; forcing low coupling (interfaces, DI) usually clumps related logic together, raising cohesion.
Don't over-decouple. Interfaces, indirection, and events add ceremony. A one-off internal helper doesn't need an interface. Apply the effort where change is likely (volatile dependencies like payment providers, external systems) and stay concrete where things are stable. Premature abstraction is its own smell.
Cohesion is roughly "intra-module quality," coupling is "inter-module quality" — you want to maximise the former and minimise the latter, and both are proxies for the real goal: changes stay local and modules are independently testable and replaceable.
Summary + common mistakes

Aim for modules that do one thing (high cohesion) and know as little as possible about each other (low coupling). Achieve it with SRP, interface-based dependency injection, encapsulation/Law of Demeter, narrow contracts, and event-driven seams. Common mistakes: god classes / Utils dumping grounds (low cohesion); reaching into other objects' internals and passing control flags (high coupling); over-abstracting stable code (needless indirection). A strong answer ties both back to the practical payoff — localised changes, easy testing, safe substitution — and gives a real refactor rather than only definitions.

**Comparison:**

```
Property	Cohesion	Coupling
Scope	Inside one module	Between modules
You want	HIGH	LOW
Best form	Functional (one task)	Data coupling (pass only needed values)
Worst form	Coincidental (Utils grab-bag)	Content coupling (reach into internals)
Bad symptom	God class / many reasons to change	Shotgun surgery / can't test in isolation
Main lever	Single Responsibility Principle	Interfaces + Dependency Injection
Payoff	Understandable, focused, reusable modules	Independent change, easy tests, safe swaps
```

**Code:**

```

// BAD: low cohesion (5 responsibilities) + high coupling (concrete deps, no seams) class OrderService { void placeOrder(Order o) { if (o.items().isEmpty()) throw new IllegalArgumentException(); // validation var conn = DriverManager.getConnection("jdbc:postgresql://...");// direct DB // ...raw SQL insert... double tax = o.total() * 0.18; // tax logic Stripe.charge(o.card(), o.total() + tax); // concrete gateway SmtpClient.send(o.email(), "<html>Thanks...</html>"); // email formatting } }

CODE
Copy

```java
// GOOD: each collaborator is cohesive; OrderService depends on interfaces (low coupling)
interface OrderRepository { void save(Order o); }
interface PaymentGateway  { void charge(Card c, long amountPaise); }
interface Notifier        { void notifyPlaced(Order o); }

class OrderService {                     // orchestrator only — one reason to change
    private final OrderValidator validator;
    private final OrderRepository repo;      // injected abstraction
    private final TaxCalculator tax;
    private final PaymentGateway payment;    // swap Stripe->Razorpay w/o touching this
    private final Notifier notifier;

    OrderService(OrderValidator v, OrderRepository r, TaxCalculator t,
                 PaymentGateway p, Notifier n) {
        this.validator = v; this.repo = r; this.tax = t;
        this.payment = p; this.notifier = n;
    }

    void placeOrder(Order o) {
        validator.validate(o);
        long total = o.subtotalPaise() + tax.forOrder(o);
        payment.charge(o.card(), total);
        repo.save(o);
        notifier.notifyPlaced(o);        // could be an event instead, fully decoupling consumers
    }
}
```

**Example input / output:**

```text
Testing payoff of the refactor (why low coupling matters in practice):

Before (high coupling):
  test placeOrder  ->  needs a live Postgres + Stripe network + SMTP server -> slow, flaky, unrunnable in CI unit stage.

After (low coupling via injected interfaces):
  var svc = new OrderService(validator, mockRepo, tax, mockGateway, mockNotifier);
  svc.placeOrder(sampleOrder);
  verify(mockGateway).charge(card, 11800);  // subtotal 10000 + 18% tax 1800
  verify(mockRepo).save(sampleOrder);
  verify(mockNotifier).notifyPlaced(sampleOrder);
  -> pure in-memory unit test, milliseconds, deterministic.

Swap a provider (Stripe -> Razorpay): implement RazorpayGateway (PaymentGateway),
change only the wiring/DI config. OrderService untouched -> change stayed local.
```

**Flow:**

1. LOW_COUPLING_HIGH_COHESION [Good: focused pieces behind interfaces]
2. impl
3. impl
4. impl
5. OrderService orchestrates
6. OrderValidator
7. {OrderRepository iface}
8. {PaymentGateway iface}
9. {Notifier iface}
10. PostgresOrderRepository
11. StripeGateway
12. EmailNotifier
13. HIGH_COUPLING_LOW_COHESION [Bad: tangled god class]
14. OrderService: validate + SQL +tax + Stripe + SMTP
15. (Postgres, concrete)
16. Stripe SDK, concrete
17. SMTP, concrete

