---
type: "practice"
difficulty: "Intermediate"
---

# Explain the Strategy pattern with an example like different payment methods.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Over-engineering vs Simplicity](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Explain the Strategy behavioral pattern and demonstrate it with a payment-methods example (a natural fit given the fintech/e-commerce tag list — Razorpay, PhonePe, Paytm, CRED, Flipkart). The interviewer is probing whether I can (a) articulate the intent — encapsulate interchangeable algorithms behind a common interface and select at runtime — and (b) show why it beats a growing if/else/switch on payment type, i.e. Open/Closed Principle. At "intermediate experience" level, they want practitioner framing: how it's wired (DI/registry), how it's tested, and its trade-offs versus alternatives.

#### Assumptions
/clarifications I'd state: "Payment method" = card, UPI, wallet, net-banking, etc., each with its own processing logic but the same "pay this amount" contract. I'll assume a typed language (Java/TypeScript) and a context object (a PaymentProcessor or checkout service) that holds/selects a strategy. I'll keep provider I/O abstracted (no real SDK calls) so the pattern stays the focus.

Plan: (1) define Strategy from first principles, (2) name the smell it removes, (3) build the payment example (interface + concrete strategies + context + selection), (4) show how it's registered/selected at runtime, (5) trade-offs and comparison to State/Template Method, (6) common mistakes.

Strategy — first principles

The Strategy pattern defines a family of algorithms, encapsulates each one in its own class behind a common interface, and makes them interchangeable at runtime. The client (a "context") is written against the interface, so the behavior it uses can be swapped without changing the client's code.

GoF intent: "Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it."

Core moves:

A Strategy interface declaring the operation (e.g., pay(amount)).
Multiple Concrete Strategies, each implementing the operation differently (CardStrategy, UpiStrategy, WalletStrategy...).
A Context that holds a reference to a strategy and delegates to it, plus a way to select/inject the right strategy (constructor injection, a setter, or a registry/factory keyed by payment type).

This favors composition over inheritance: instead of subclassing the context for each behavior, you compose it with a strategy object and can even change it at runtime.

The smell it removes

Without Strategy, payment handling tends to become one fat method:

CODE
if (type == CARD)      { /* card logic */ }
else if (type == UPI)  { /* upi logic */ }
else if (type == WALLET){ /* wallet logic */ }
// ... grows with every new method

Problems: the method grows unboundedly, every new payment type modifies existing tested code (violating Open/Closed), unrelated logics are tangled in one place (hard to test in isolation), and duplication creeps across similar branches. Strategy replaces the conditional with polymorphic dispatch: adding a payment method means adding a class, not editing a switch.

Payment-methods example
Strategy interface: PaymentStrategy { PaymentResult pay(Money amount, PaymentContextData ctx); }
Concrete strategies: CreditCardPayment, UpiPayment, WalletPayment, NetBankingPayment — each encapsulates its own validation, provider call, and result mapping.
Context: a CheckoutService/PaymentProcessor that is given a strategy and calls strategy.pay(...). It doesn't know or care which concrete method it holds.
Selection: a PaymentStrategyFactory/registry maps a PaymentMethod enum or string (from the user's checkout choice) to the correct strategy instance. In Spring this is often a Map<String, PaymentStrategy> auto-populated by bean name, so a new strategy just gets a @Component and appears in the map — zero edits to the selector.

At runtime: user picks "UPI" → factory returns UpiPayment → checkout.setStrategy(upi) (or inject) → checkout.checkout(amount) delegates to upi.pay(...). Switch to "Card" for the next order by swapping the strategy — the context is unchanged.

Why this is the right design here
Open/Closed: new payment methods = new classes; existing code untouched.
Single Responsibility: each strategy owns exactly one payment method's logic; unit tests are focused and independent.
Runtime flexibility: the same checkout can use different methods per order/user; you can even A/B or feature-flag strategies.
Testability: inject a fake/mock strategy into the context; test each strategy in isolation.
Reduced complexity: dispatch is table/polymorphism-driven, not a mega-conditional.

#### Trade-offs
and alternatives
More classes/objects — for only two rarely-changing branches, a simple conditional or a couple of lambdas may be simpler; don't over-apply. In modern languages a Strategy is often just a function/lambda (Function<Money, Result>) rather than a full class — same pattern, less ceremony.
Client must know strategies exist to pick one — the selection logic (factory/registry) still lives somewhere; Strategy relocates it to one small place rather than scattering it.
Shared data: strategies may need context data (card token, VPA, user id); pass it via method params or a context DTO to keep strategies stateless and reusable/singleton-safe.

Comparison to neighbors:

State pattern has near-identical structure but different intent: State transitions between behaviors driven by internal state and the objects often know about each other; Strategy's algorithms are independent and chosen by the client.
Template Method varies steps via inheritance (override hooks in a fixed skeleton); Strategy varies the whole algorithm via composition and can change at runtime. If payment methods share a common skeleton (validate → charge → record) with only a couple of differing steps, Template Method or a hybrid (skeleton + injected sub-strategies) can be cleaner.
Factory complements Strategy: the factory creates/selects the strategy; Strategy executes it.

#### Common mistakes
/ follow-up probes
Leaving the if/else in the factory and calling it "Strategy." A single lookup (map) is fine; a growing conditional that does the work is not. The point is to remove behavior branching from the hot path.
Stateful strategies shared as singletons causing race conditions — keep strategies stateless; pass per-request data as arguments.
"Isn't this just polymorphism?" Yes — Strategy is a disciplined use of polymorphism to make algorithms pluggable and selectable at runtime; the value is the naming/structure and OCP guarantee, plus a clear selection seam.

#### Follow-up
"How do you add a new method with zero edits to existing code?" → register a new @Component strategy keyed by its method; the Map-based selector picks it up automatically.
Summary — what a strong answer emphasizes

Strategy encapsulates each payment method's algorithm behind a common pay() interface and lets the checkout context swap methods at runtime by holding a strategy reference. It replaces a brittle, ever-growing if/else/switch with polymorphic dispatch, delivering Open/Closed extensibility (new method = new class), single-responsibility testability, and runtime flexibility. Use a factory/registry to select the strategy from the user's choice; keep strategies stateless. Apply it when you have multiple interchangeable behaviors that vary or grow; skip the full class ceremony (use lambdas or a plain conditional) when the branches are few and stable.

**Comparison:**

```
Concern	if/else on payment type	Strategy pattern
Add a new method	Edit & re-test existing method (violates OCP)	Add a new class; existing code untouched
Testability	One fat method, hard to isolate	Each strategy tested independently
Responsibility	All logics tangled together	One method per class (SRP)
Runtime switch	Re-run the conditional	Swap the strategy reference
Best when	2–3 stable branches	Many/growing interchangeable behaviors
Pattern	Structure	Intent difference vs Strategy
Strategy	Context composes an algorithm object	Client picks interchangeable algorithm at runtime
State	Same shape; states may transition each other	Behavior driven by internal state transitions
Template Method	Inheritance, override hook steps	Fixed skeleton, vary steps (compile-time) not whole algo
Factory	Creates/returns objects	Selects/creates the strategy; doesn't execute it
```

**Code:**

```java
// 1) Strategy interface — the common contract
public interface PaymentStrategy {
    PaymentResult pay(Money amount, PaymentContextData data);
    String method();                 // "CARD", "UPI", "WALLET", ...
}

// 2) Concrete strategies — each encapsulates ONE payment method's algorithm
public class CreditCardPayment implements PaymentStrategy {
    public String method() { return "CARD"; }
    public PaymentResult pay(Money amount, PaymentContextData d) {
        // validate card token, call card gateway, map response...
        return PaymentResult.ok("CARD charged " + amount + " token=" + d.get("cardToken"));
    }
}
public class UpiPayment implements PaymentStrategy {
    public String method() { return "UPI"; }
    public PaymentResult pay(Money amount, PaymentContextData d) {
        // create UPI collect request to VPA, poll/callback...
        return PaymentResult.ok("UPI collect " + amount + " vpa=" + d.get("vpa"));
    }
}
public class WalletPayment implements PaymentStrategy {
    public String method() { return "WALLET"; }
    public PaymentResult pay(Money amount, PaymentContextData d) {
        // debit wallet balance...
        return PaymentResult.ok("WALLET debit " + amount);
    }
}

// 3) Context — depends only on the interface; behavior is injected/selected
public class PaymentProcessor {
    private PaymentStrategy strategy;                 // composition, not inheritance
    public PaymentProcessor(PaymentStrategy s) { this.strategy = s; }
    public void setStrategy(PaymentStrategy s) { this.strategy = s; }   // swap at runtime
    public PaymentResult checkout(Money amount, PaymentContextData data) {
        return strategy.pay(amount, data);            // polymorphic dispatch, no if/else
    }
}

// 4) Selection — registry/factory keyed by method (NO growing switch in the hot path)
public class PaymentStrategyFactory {
    private final Map<String, PaymentStrategy> byMethod = new HashMap<>();
    public PaymentStrategyFactory(List<PaymentStrategy> strategies) {
        for (PaymentStrategy s : strategies) byMethod.put(s.method(), s);
    }
    public PaymentStrategy of(String method) {
        PaymentStrategy s = byMethod.get(method);
        if (s == null) throw new IllegalArgumentException("Unsupported method: " + method);
        return s;   // adding a new method = add a strategy to the list; this code is untouched
    }
}
```

**Example input / output:**

```text
var factory = new PaymentStrategyFactory(List.of(
        new CreditCardPayment(), new UpiPayment(), new WalletPayment()));

// User chose UPI at checkout:
var proc = new PaymentProcessor(factory.of("UPI"));
proc.checkout(Money.inr(1499), data.with("vpa","ann@okhdfc"));
// -> PaymentResult{ok=true, msg="UPI collect INR 1499.00 vpa=ann@okhdfc"}

// Same processor, switch method at runtime (retry via card):
proc.setStrategy(factory.of("CARD"));
proc.checkout(Money.inr(1499), data.with("cardToken","tok_1a2b3c"));
// -> PaymentResult{ok=true, msg="CARD charged INR 1499.00 token=tok_1a2b3c"}

// Unknown method:
factory.of("CRYPTO"); // -> IllegalArgumentException: Unsupported method: CRYPTO
```

**Data:**

```json
// PaymentContextData examples passed per request (keeps strategies stateless)
{ "method": "CARD",   "cardToken": "tok_1a2b3c", "amount": 1499.00, "currency": "INR" }
{ "method": "UPI",    "vpa": "ann@okhdfc",       "amount": 1499.00, "currency": "INR" }
{ "method": "WALLET", "walletId": "w_998877",     "amount": 1499.00, "currency": "INR" }
```

**Flow:**

1. impl
2. impl
3. impl
4. inject chosen strategy
5. checkout ->strategy.pay(amount,data)
6. User selects method atcheckout
7. PaymentStrategyFactory.of(method)
8. {PaymentStrategy interface}
9. CreditCardPayment
10. UpiPayment
11. WalletPayment
12. PaymentProcessor context
13. PaymentResult

