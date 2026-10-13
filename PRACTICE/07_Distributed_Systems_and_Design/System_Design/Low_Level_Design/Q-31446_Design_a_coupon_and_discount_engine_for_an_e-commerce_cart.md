---
type: "practice"
difficulty: "Deep"
---

# Design a coupon and discount engine for an e-commerce cart.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rule Engines and Pricing](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Rule_Engines_and_Pricing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Design the low-level structure of a coupon and discount engine that a cart calls at checkout — the data model, the rule/condition model, how a discount is applied to a cart, and the API. This is a Flipkart/Amazon/Swiggy LLD question, so the interviewer wants clean object modeling, extensibility (open/closed), correctness of the amount math, and validation/edge cases — not distributed-systems hand-waving.

What they're probing (deep-understanding level):

Can you model many kinds of promotions (flat, percentage, BOGO, free shipping, bank/card offers, tiered/threshold) behind one clean abstraction so adding a new promo type needs no rewrite?
Do you separate eligibility (conditions) from effect (actions)?
Do you handle validation, limits (per-user, global, expiry, usage count), and idempotent redemption?
Is the money math correct (rounding, max-discount caps, applying to the right line items, tax/shipping order)?

Clarifying questions I'd ask: Coupon-code driven or auto-applied campaigns too? Cart-level or line-item discounts? Do multiple discounts stack (I'll flag this as a separate concern — see follow-up)? Currency/rounding rules? For this answer I assume: both code-based and auto campaigns, discounts can target the whole cart or specific line items, and I'll design for single best coupon here (stacking is qid 885's territory) while making the model stacking-ready.

Plan: requirements → core domain model (Coupon, Rule = Condition + Action via Strategy) → the apply algorithm with correct math → API → validation/limits → persistence → summary + edge cases.

1. Requirements

Functional: validate a coupon code against a cart + user; compute the discount; support flat-off, percent-off (with max cap), free shipping, BOGO/buy-X-get-Y, tiered thresholds ("₹100 off over ₹999"), and category/brand/first-order targeting; enforce expiry, usage limits, min-cart-value, applicable products. Non-functional: extensible (new promo type = new class, no edits elsewhere), deterministic and idempotent, fast (checkout is latency-sensitive), auditable (why did/didn't it apply).

2. Domain model — Condition + Action behind a Strategy

The key design move: a coupon is a set of Conditions (is it eligible?) + an Action (what discount to give). Both are polymorphic interfaces (Strategy pattern). This is what makes the engine open/closed — adding "free shipping over ₹500 for Prime users" is a new Condition + a new Action, zero changes to the evaluator.

CODE
Coupon
  code, type(PUBLIC/PRIVATE/AUTO), validFrom, validTo, active
  conditions: List<Condition>      // ALL must pass (AND); OR via composite
  action: DiscountAction           // the effect
  limits: UsageLimits              // global, per-user, per-day
  priority, stackable               // for the stacking engine (qid 885)

interface Condition   { boolean isSatisfied(CartContext ctx); String reason(); }
interface DiscountAction { DiscountResult apply(CartContext ctx); }

Concrete conditions: MinCartValueCondition, FirstOrderCondition, CategoryInCartCondition, PaymentMethodCondition (card/bank offers), UserSegmentCondition, TimeWindowCondition. Concrete actions: FlatDiscount, PercentageDiscount(maxCap), FreeShippingDiscount, BuyXGetYDiscount, TieredDiscount. Each is independently unit-testable.

3. The apply algorithm (correctness lives here)
CODE
applyCoupon(cart, user, code):
  coupon = repo.findActiveByCode(code)                 # 404 if none
  guard(coupon.active and inWindow(now))               # EXPIRED
  guard(withinUsageLimits(coupon, user))               # LIMIT_REACHED
  for c in coupon.conditions:
      if not c.isSatisfied(ctx): reject(c.reason())    # NOT_ELIGIBLE(+why)
  result = coupon.action.apply(ctx)                    # compute amount
  result.discount = min(result.discount, cart.eligibleSubtotal)  # never negative total
  return result   # {couponCode, discountAmount, appliedTo:[lineIds], freeShipping}

Money-math rules that separate a strong answer:

Compute on the eligible subtotal (only the line items the coupon targets), not the grand total, unless it's a cart-wide coupon.
Percentage with cap: min(round(subtotal * pct/100), maxCap). Missing the cap is the classic bug.
Rounding: round to currency minor units at the discount level, consistently (banker's or half-up — pick one and document). Never let floating point pick.
Order of operations with tax/shipping: apply product discounts to pre-tax line totals; recompute tax on the discounted amount; free-shipping affects the shipping charge only. State this order explicitly.
Cap the discount so the payable never goes below zero (or below a configured floor).
4. API

POST /cart/{id}/apply-coupon {code} → returns updated cart pricing with a discounts[] breakdown and, on failure, a machine-readable reason. DELETE /cart/{id}/coupon/{code} to remove. Redemption is finalized at order placement, not at apply-time, and is idempotent (idempotency key = orderId+couponCode) so a retried checkout doesn't burn two uses.

5. Validation, limits & redemption
Usage limits: global count, per-user count, per-day — enforced atomically at redemption (UPDATE ... WHERE remaining > 0 returning affected rows, or a Redis counter) to avoid over-redemption under concurrency. Apply-time is a soft check; redemption-time is the hard gate.
Applicability: min cart value, included/excluded categories/brands/SKUs, new-user only, region.
Auditability: every apply/reject returns the failing condition's reason() so support and the UI ("Add ₹200 more to use SAVE100") are driven by the same engine.
Redemption record links coupon↔user↔order for idempotency and analytics.
6. Persistence (sketch)

coupons (code, action config JSON, priority, stackable, windows, limits), coupon_conditions (or condition config embedded as JSON for flexibility), coupon_redemptions (unique on order_id+code). Store action/condition params as typed JSON so marketing can create campaigns without a schema migration; validate against a schema on write.

7. Summary — what a strong answer emphasizes
Separate eligibility (Conditions) from effect (Actions), both as Strategy interfaces → open/closed extensibility (the single most important point).
Correct money math: eligible-subtotal targeting, percentage caps, deterministic rounding, explicit tax/shipping order, never-negative total.
Two-phase redemption: soft-validate at apply, atomic hard-gate usage limits at order placement, idempotent by (order, code).
Reason-carrying validation so UI/support reuse the same rules.
Model is stacking-ready (priority + stackable flags) even though single-coupon is the default — which sets up the follow-up.

#### Common mistakes
giant if/else per coupon type (unmaintainable); forgetting the percentage cap; float rounding drift; checking usage limits only at apply-time (over-redemption race); applying discount to grand total including tax; not making redemption idempotent. Follow-up probes: stacking/priority (qid 885), fraud/abuse (velocity limits), A/B campaign rollout, and caching active coupons for checkout latency.

**Comparison:**

```
Promo type	Condition(s)	Action	Key math detail
Flat off	min cart value	FlatDiscount(amt)	min(amt, subtotal)
Percent off	category/user	PercentageDiscount(pct, cap)	cap the percentage, round HALF_UP
Free shipping	min value / segment	FreeShippingDiscount	zero out shipping only
BOGO	qty of SKU >= X	BuyXGetYDiscount	discount = price of Y cheapest
Tiered	threshold bands	TieredDiscount(bands)	pick highest satisfied band
Bank/card offer	payment method	PercentageDiscount(cap)	evaluated at payment step
```

**Code:**

```

// Strategy interfaces — the extensibility backbone interface Condition { boolean isSatisfied(CartContext ctx); String reason(); } interface DiscountAction { DiscountResult apply(CartContext ctx); }

record DiscountResult(BigDecimal discount, List<String> appliedLineIds, boolean freeShipping) {}

// Example condition class MinCartValueCondition implements Condition { private final BigDecimal min; MinCartValueCondition(BigDecimal min) { this.min = min; } public boolean isSatisfied(CartContext c) { return c.eligibleSubtotal().compareTo(min) >= 0; } public String reason() { return "Add more items to reach cart minimum of " + min; } }

// Percentage discount WITH cap and deterministic rounding class PercentageDiscount implements DiscountAction { private final BigDecimal pct; private final BigDecimal maxCap; PercentageDiscount(BigDecimal pct, BigDecimal maxCap) { this.pct = pct; this.maxCap = maxCap; } public DiscountResult apply(CartContext c) { BigDecimal raw = c.eligibleSubtotal() .multiply(pct).divide(BigDecimal.valueOf(100), 2, RoundingMode.HALF_UP); BigDecimal capped = maxCap != null ? raw.min(maxCap) : raw; BigDecimal safe = capped.min(c.eligibleSubtotal()); // never below zero return new DiscountResult(safe, c.eligibleLineIds(), false); } }

// Engine: evaluate one coupon against the cart class CouponEngine { ApplyResult apply(Coupon coupon, CartContext ctx) { if (!coupon.isActiveNow()) return ApplyResult.reject("EXPIRED"); if (!limitStore.hasQuota(coupon, ctx.userId())) return ApplyResult.reject("LIMIT_REACHED"); for (Condition cond : coupon.conditions()) if (!cond.isSatisfied(ctx)) return ApplyResult.reject("NOT_ELIGIBLE", cond.reason()); DiscountResult r = coupon.action().apply(ctx); return ApplyResult.ok(coupon.code(), r); } }

CODE
Copy

```sql
-- Atomic usage-limit gate at REDEMPTION (prevents over-redemption under concurrency)
UPDATE coupons
   SET remaining_uses = remaining_uses - 1
 WHERE code = :code AND remaining_uses > 0;   -- rows_affected=0 => exhausted, reject
```

**Example input / output:**

```text
Cart: fashion items subtotal ₹1500, user first-time, code=SAVE200

Conditions:
  MIN_CART_VALUE(999): 1500 >= 999            -> pass
  CATEGORY_IN_CART(fashion): present          -> pass
Action PERCENTAGE(20, cap 200):
  raw   = 1500 * 20% = 300.00
  cap   = min(300, 200) = 200.00
  final = min(200, 1500) = 200.00

Response 200:
{ "couponCode":"SAVE200", "discount":200.00, "appliedTo":["li_1","li_2"], "payable":1300.00 }

--- failure case: subtotal ₹700 ---
MIN_CART_VALUE(999): 700 >= 999 -> FAIL
Response 422: { "error":"NOT_ELIGIBLE", "reason":"Add more items to reach cart minimum of 999" }
```

**Data:**

```json
// A coupon stored with typed condition/action config (marketing-editable, no migration)
{
  "code": "SAVE200",
  "type": "PUBLIC",
  "validFrom": "2026-07-01T00:00:00Z",
  "validTo":   "2026-07-31T23:59:59Z",
  "priority": 10,
  "stackable": false,
  "conditions": [
    { "kind": "MIN_CART_VALUE", "min": 999 },
    { "kind": "CATEGORY_IN_CART", "categories": ["fashion"] }
  ],
  "action": { "kind": "PERCENTAGE", "pct": 20, "maxCap": 200 },
  "limits": { "globalUses": 100000, "perUser": 1, "perDay": 5000 }
}
```

**Flow:**

1. not found
2. no
3. yes
4. no
5. yes
6. any fail
7. all pass
8. at order placement
9. POST apply-coupon
10. Load active coupon by code
11. 404 INVALID_CODE
12. Active & in window?
13. 422 EXPIRED
14. Usage quota left?
15. 422 LIMIT_REACHED
16. Evaluate ALL conditions
17. 422 NOT_ELIGIBLE + reason
18. action.apply -> raw discount
19. Cap: min discount, eligiblesubtotal
20. Return pricing breakdown
21. Atomic decrement quota +idempotent redemption record

