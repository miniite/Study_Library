---
type: "practice"
difficulty: "Deep"
---

# How does the Strategy pattern differ from a simple if-else chain? Why prefer it?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Over-engineering vs Simplicity](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The interviewer is asking me to contrast the Strategy design pattern with a plain if-else / switch chain that selects behavior at runtime. On the surface both "pick one of N behaviors based on some condition," so the real probe is: do you understand why the extra indirection of Strategy is worth it, and can you name the concrete costs it removes? A strong answer must (1) define Strategy precisely, (2) show the exact if-else it replaces, (3) tie the difference to design principles (Open/Closed, Single Responsibility, DIP), and (4) be honest about when if-else is actually the right call. This is a "deep understanding" LLD question for SDE-2/3 and above, so I should reason from first principles, use a realistic example (payment/shipping/pricing), and discuss trade-offs and testability — not just recite the GoF definition.

#### Assumptions
/ clarifiers I'd state: the branches represent interchangeable algorithms for one job (e.g., "how to compute discount", "how to charge a payment") — not unrelated logic; the set of behaviors is expected to grow or vary independently; and we care about testability and team velocity.

First principles: what problem are we solving?

Imagine an e-commerce checkout that computes a discount. A naive version:

CODE
double discount(Order o, String type) {
    if (type.equals("PERCENTAGE")) { ... }
    else if (type.equals("FLAT")) { ... }
    else if (type.equals("BOGO")) { ... }
    else if (type.equals("LOYALTY")) { ... }
    else throw ...;
}

Each else if embeds a different algorithm inside one method. The if-else is not the problem by itself — the problem is that this method now has many reasons to change and many responsibilities. Every new coupon type edits this method; the method knows the internals of all algorithms; you cannot test BOGO logic without dragging the whole selector in; and the same type string check tends to get duplicated in three other places (validation, display, refund).

What Strategy actually is

Strategy defines a family of interchangeable algorithms, encapsulates each one behind a common interface, and lets the caller (the "context") swap them at runtime. The context holds a reference to the interface, not to a concrete branch. Concretely:

DiscountStrategy interface with one method apply(Order).
One class per algorithm: PercentageDiscount, FlatDiscount, BogoDiscount, LoyaltyDiscount.
The Checkout context is given a DiscountStrategy (constructor/setter injection) and just calls strategy.apply(order). It never branches on type.

Selection moves to one place — a factory or a Map<CouponType, DiscountStrategy>. The rest of the code depends only on the abstraction.

The concrete differences (this is the heart of the answer)

1. Open/Closed Principle. With if-else, adding BankOfferDiscount means editing the working discount() method — you risk breaking existing branches and must re-test the whole method. With Strategy, you add a new class and register it; existing code is untouched. Open for extension, closed for modification.

2. Single Responsibility. Each strategy class owns exactly one algorithm and has one reason to change. The if-else version has one god-method that changes for every algorithm — a magnet for merge conflicts when a team of ten touches checkout.

3. Testability. You can unit-test BogoDiscount in isolation with no branching context. The if-else forces you to reach the branch via the right input string every time and share setup across unrelated cases.

4. Runtime pluggability & configuration. Strategy can be chosen from config, a feature flag, an A/B experiment bucket, a DB row, or a plugin loaded at startup — because it's just an object you inject. An if-else chain is compiled in; you cannot add a merchant-specific pricing rule without a code deploy.

5. Eliminating duplicated conditionals. The same switch(type) tends to appear in validate, apply, describe, refund. Strategy folds all of that behavior onto the object, so type is resolved to an object once.

6. Cyclomatic complexity / readability. The context method goes from N branches to one line. Complexity doesn't vanish — it's distributed into cohesive classes instead of concentrated in one method, which is easier to reason about and to navigate by name.

Why prefer it — and when NOT to

Prefer Strategy when: the branches are genuinely interchangeable algorithms for one operation; the set grows over time; you want to select behavior from config/flags/experiments; or the branches are individually complex and deserve their own tests. That covers payment gateways (Razorpay/Stripe/UPI), shipping-cost calculators (Uber surge, Swiggy delivery fee), sorting/pricing/tax rules, retry/backoff policies, and file exporters.

Be honest about the cost. Strategy adds classes and indirection. If you have 2–3 branches that are one line each, rarely change, and are never selected dynamically, a switch is simpler and more readable — over-applying Strategy is its own smell (needless indirection, harder to trace). The judgment call: does the behavior vary independently and grow? If yes → Strategy; if it's a small fixed set → if-else is fine. Also note if-else still lives somewhere in Strategy — in the factory that maps a key to an object — but it's isolated to one small, easily tested place instead of scattered through business logic.

Modern nuance. In languages with first-class functions/lambdas, a lightweight Strategy is just a Map<Key, Function> or passing a lambda — same pattern, far less ceremony. That's often the pragmatic middle ground between a raw if-else and a full class hierarchy.

Summary — what a strong answer emphasizes

Both replace runtime branching, but Strategy trades a small amount of indirection for extensibility without modification (OCP), one-responsibility-per-class (SRP/testability), and runtime/config-driven selection (DIP: depend on the interface). The if-else concentrates many algorithms and many reasons-to-change into one method; Strategy distributes them into cohesive, independently testable units and resolves the choice in exactly one factory/map. Prefer Strategy when behaviors are interchangeable, numerous, growing, or config-selected; keep the if-else when the set is tiny, static, and trivial. The mature signal is knowing both directions — and that even Strategy keeps one small, contained conditional in its factory.

Common follow-up probes: "Where does the if-else go now?" (the factory/registry). "Strategy vs State?" (State transitions between behaviors and the object drives its own change; Strategy is chosen by the client and doesn't self-transition). "How do you pass shared context to strategies?" (method parameters or a context object, avoiding stateful strategies so they stay reusable/singletons).

**Comparison:**

```
Aspect	if-else / switch chain	Strategy pattern
Adding a behavior	Edit the shared method (risky)	Add a class + register (OCP)
Responsibility	One god-method owns all algorithms	One algorithm per class (SRP)
Testability	Reach branch via right input; shared setup	Test each strategy in isolation
Runtime selection	Compiled in; needs a deploy	Inject from config/flag/DB/experiment
Duplicated conditionals	switch(type) repeats across code	Resolved to an object once, in factory
Complexity of context	Grows with N branches	Constant (one delegating call)
Best when	2–3 tiny, static branches	Many, growing, interchangeable, config-selected
Cost	None extra (fewer classes)	More classes + indirection
```

**Code:**

```

// --- The abstraction: a family of interchangeable algorithms --- public interface DiscountStrategy { Money apply(Order order); }

// --- One class per algorithm (SRP: one reason to change each) --- public final class PercentageDiscount implements DiscountStrategy { private final double pct; public PercentageDiscount(double pct) { this.pct = pct; } public Money apply(Order o) { return o.total().times(pct / 100.0); } }

public final class FlatDiscount implements DiscountStrategy { private final Money amount; public FlatDiscount(Money amount) { this.amount = amount; } public Money apply(Order o) { return Money.min(amount, o.total()); } }

public final class NoDiscount implements DiscountStrategy { // null-object, no null checks public Money apply(Order o) { return Money.ZERO; } }

// --- The context depends ONLY on the interface (no branching) --- public final class Checkout { private DiscountStrategy strategy = new NoDiscount(); public void setStrategy(DiscountStrategy s) { this.strategy = s; } // swap at runtime public Money finalPrice(Order o) { return o.total().minus(strategy.apply(o)); // one line, no if-else } }

// --- Selection isolated to ONE place: the factory/registry --- public final class DiscountFactory { private static final Map<CouponType, DiscountStrategy> REGISTRY = Map.of( CouponType.PERCENTAGE, new PercentageDiscount(10), CouponType.FLAT, new FlatDiscount(Money.of(150)), CouponType.NONE, new NoDiscount() ); public static DiscountStrategy forType(CouponType t) { return REGISTRY.getOrDefault(t, new NoDiscount()); } }

CODE
Copy

```java
// The if-else version this replaces (many reasons to change in one method):
Money discount(Order o, CouponType type) {
    if (type == CouponType.PERCENTAGE) return o.total().times(0.10);
    else if (type == CouponType.FLAT)  return Money.min(Money.of(150), o.total());
    else if (type == CouponType.NONE)  return Money.ZERO;
    else throw new IllegalArgumentException("unknown coupon");
}
JAVA
Copy
// Lightweight Strategy in a language with first-class functions (less ceremony):
Map<CouponType, Function<Order, Money>> strategies = Map.of(
    CouponType.PERCENTAGE, o -> o.total().times(0.10),
    CouponType.FLAT,       o -> Money.min(Money.of(150), o.total())
);
Money d = strategies.getOrDefault(type, o -> Money.ZERO).apply(order);
```

**Example input / output:**

```

Worked example — adding a NEW behavior, if-else vs Strategy:

CODE
Copy
Scenario: product adds "BANK_OFFER: 5% off, capped at ₹500 on HDFC cards"

if-else path:
  1. OPEN the shared discount() method (working code).
  2. INSERT a new "else if (type == BANK_OFFER)" branch.
  3. Re-test EVERY branch (edit touched shared code).  -> OCP violated.

Strategy path:
  1. ADD file BankOfferDiscount implements DiscountStrategy.
  2. REGISTER CouponType.BANK_OFFER -> new BankOfferDiscount() in the factory map.
  3. Existing PercentageDiscount/FlatDiscount classes + Checkout: UNTOUCHED.
     Unit-test only the new class.                     -> OCP satisfied.

Runtime selection example:
  input : order.total = ₹2000, couponType = PERCENTAGE(10)
  factory.forType(PERCENTAGE) -> PercentageDiscount(10)
  checkout.finalPrice(order)  -> 2000 - 200 = ₹1800     (context never branched)
```

**Flow:**

1. delegates (no if-else)
2. builds/registers (branchinglives HERE, once)
3. Checkout
4. -DiscountStrategy strategy
5. +setStrategy(DiscountStrategy)
6. +finalPrice(Order) : Money
7. «interface»
8. DiscountStrategy
9. +apply(Order) : Money
10. PercentageDiscount
11. FlatDiscount
12. NoDiscount
13. DiscountFactory
14. +forType(CouponType) : DiscountStrategy

