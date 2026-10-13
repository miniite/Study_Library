---
type: "practice"
difficulty: "Intermediate"
---

# Design a shopping cart. What operations does it expose and how do you model line items?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Classic LLD Problems](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Classic_LLD_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: "Design a shopping cart" is a classic LLD (low-level design) warm-up. The interviewer wants to see whether you can take a familiar consumer feature and produce a clean object model with the right entities, responsibilities, operations (the public API), invariants, and edge cases — without drifting into full distributed-system design. Two specific sub-prompts are pinned down for me: (1) what operations does the cart expose and (2) how do you model line items. So I must be concrete about both the class diagram and the method surface.

What the interviewer is probing for at "intermediate experience":

Do you separate Cart, LineItem (CartItem), and Product cleanly, or do you conflate them?
Do you understand that a line item is not a product — it captures product identity + quantity + a price snapshot + variant/options?
Do you handle idempotent add (adding an existing product increments quantity instead of duplicating a row)?
Do you think about pricing: is price stored on the product or snapshotted on the item? Money precision? Discounts/coupons?
Concurrency & persistence: guest vs logged-in cart, merge on login, stock re-validation, price drift at checkout.

Clarifying assumptions I'll state: single-currency B2C cart; a product can have variants (size/color) so the true cart key is (productId, variantId); the cart is persisted (survives sessions) and eventually converts to an Order; the cart itself does NOT reserve inventory (reservation happens at checkout) — this is the mainstream e-commerce model and I'll call out the alternative.

First principles: what a cart actually is

A shopping cart is a mutable collection of intents to buy, owned by a user (or an anonymous session), that we hold until checkout. It is deliberately not an order: nothing is charged, and (in the common model) nothing is reserved. That framing drives every decision below.

The three core entities
Product (catalog domain) — the sellable thing: id, name, basePrice, and it owns Variants (variantId, sku, attributes like size/color, per-variant price/stock). The cart references products; it does not own them.
CartItem / LineItem — the heart of the question. A line item captures the decision to buy N of a specific variant at a captured price. Its fields:
productId + variantId → the identity key for de-duplication.
quantity (int ≥ 1).
unitPriceSnapshot — the price at the time it was added, so the cart total is stable and auditable even if the catalog price changes. We re-validate against live price at checkout.
selectedOptions (e.g., gift-wrap, engraving) — options that don't change SKU but affect price/fulfilment.
Derived: lineTotal = unitPriceSnapshot × quantity (+ option adjustments).
Cart — owns a set of line items keyed by (productId, variantId), plus cartId, userId (nullable for guests), timestamps, optional appliedCoupon, and currency. It exposes the operations and enforces invariants (no zero-qty rows, one row per variant).

Why price is snapshotted on the line item, not read live: if you always read the live product price, the displayed cart total silently changes under the user, which is a bad UX and an audit nightmare. Snapshotting gives a stable, explainable total; a separate re-price/validate step at checkout reconciles drift and is where you decide whether to honor the old price or prompt the user. This snapshot-vs-live distinction is the single most common thing interviewers want you to articulate.

Money handling

Never use float for money. Use integer minor units (paise/cents) or BigDecimal. All arithmetic (line totals, discounts, tax) stays in that type until display formatting.

The operations the cart exposes (the public API)

The cart's surface is small and should read like the mental model of a shopper:

addItem(productId, variantId, qty) — idempotent on the key: if that variant already exists, increment quantity (clamped to stock/limit); otherwise create a line item with a fresh price snapshot.
updateQuantity(lineItemId, qty) — set an absolute quantity; qty == 0 removes the line (a clean way to model "remove via stepper").
removeItem(lineItemId).
clear().
applyCoupon(code) / removeCoupon().
getSummary() — returns subtotal, discount, (est.) tax, shipping estimate, grand total, and item count. This is a read model, computed on demand.
merge(otherCart) — merge a guest cart into the user cart on login (sum quantities per key, re-snapshot as needed).
revalidate() — refresh stock availability and detect price drift before checkout; returns a list of changes for the UI to confirm.

Design notes that signal seniority:

addItem returning the updated line lets the frontend render without a second round-trip.
Keep totals out of stored state — compute them; storing a total field invites drift bugs. (You may cache a computed summary, but the source of truth is the items + coupon.)
Validation belongs in the Cart, not the controller: min/max qty per line, max distinct items, stock clamp.
Pricing & discounts — keep it extensible

Discounts multiply fast (percentage, flat, BOGO, category-level, coupon, membership). Don't if/else them in the cart. Use a pricing pipeline / Strategy + Chain-of-Responsibility: an ordered list of PricingRule objects each transform a PriceContext (items, user, coupon). The cart just runs the chain and reads the result. This keeps Cart closed for modification and open for new promo types — the Open/Closed principle in action.

Persistence, concurrency, guest→user
Storage: a Cart row + CartItem rows keyed on (cart_id, product_id, variant_id) with a unique constraint on that triple — the DB enforces the "one row per variant" invariant even under races.
Guest carts: keyed by an anonymous session/cookie id; on login, merge into the user's existing cart (sum quantities, honor stock/limit clamps).
Concurrency: two tabs adding the same variant should not create two rows — the unique key + an upsert (INSERT ... ON CONFLICT DO UPDATE quantity = quantity + n) makes addItem safe.
Does the cart reserve stock? In the mainstream model, no — reservation happens at checkout/payment (soft-reserve with TTL). Reserving at add-to-cart hurts availability for everyone and needs release-on-abandon. I'd call this out as a deliberate trade-off; flash-sale platforms sometimes reserve, accepting the complexity.
Abandonment: carts have a TTL / cleanup job; abandoned-cart emails read this state.
Edge cases a strong answer names
Adding a variant that's now out of stock → clamp or reject with a typed error.
Price changed since add → surface at revalidate(), let the user accept.
Max quantity per user (e.g., 2 per customer on a sale item).
Currency/locale consistency; a cart is single-currency.
Deleted/unpublished product still in cart → mark line unavailable, exclude from total.
Summary — what a strong answer emphasizes

Model three entities — Product (owns Variants), CartItem (the identity (product,variant) + quantity + price snapshot + options), and Cart (owner + invariants + operations). Make addItem idempotent on the variant key, compute totals rather than store them, snapshot price on add and re-validate at checkout, use integer money, and push discounts into an extensible pricing pipeline. Enforce the "one line per variant" invariant with a DB unique constraint so concurrent adds can't duplicate. Call out that the cart normally does not reserve inventory, and handle guest→user merge. Naming the price-snapshot-vs-live distinction, the idempotent add, and computed-not-stored totals is what separates a senior answer from a CRUD one.

#### Common mistakes
storing price only on Product (total drifts), duplicating rows per add, storing the grand total as a column (drift), using floats for money, and reserving stock at add-to-cart without any release mechanism.

Likely follow-ups: "How do you merge guest and user carts?", "Where does tax/shipping get computed?", "How do you keep the cart consistent across devices?", "What changes for a flash sale?" — all answerable directly from the model above.

**Comparison:**

```
Decision	Chosen approach	Alternative	Why chosen
Price source	Snapshot on line item at add-time	Read live product price	Stable, auditable total; re-validate at checkout
De-dup key	(productId, variantId) unique	New row per add	Idempotent add; one row per variant
Totals	Computed in getSummary()	Stored total column	Avoids drift bugs
Money type	Integer minor units / BigDecimal	float/double	No rounding errors
Stock reservation	At checkout (soft-reserve+TTL)	At add-to-cart	Keeps availability high; less complexity
Discounts	Pricing pipeline (Strategy/Chain)	if/else in Cart	Open/Closed; easy new promo types
Guest→user	merge() sums per key	Discard guest cart	Don't lose the shopper's intent
Entity	Owns	Key fields
Product	Variants	id, name, basePrice
Variant	—	variantId, sku, attributes, price, stock
CartItem	—	(productId,variantId), quantity, priceSnapshot, options
Cart	CartItems	cartId, userId?, currency, coupon
```

**Code:**

```java
// Money as minor units (paise/cents) — never float.
public record Money(long amountMinor, String currency) {
    Money plus(Money o) { require(o.currency.equals(currency)); return new Money(amountMinor + o.amountMinor, currency); }
    Money times(int q)  { return new Money(amountMinor * q, currency); }
    static Money zero(String c) { return new Money(0, c); }
}

// Identity key for de-duplication: product + variant.
public record CartItemKey(String productId, String variantId) {}

public class CartItem {
    final CartItemKey key;
    int quantity;                 // >= 1
    final Money unitPriceSnapshot;// price captured at add-time
    final Map<String,String> selectedOptions;

    CartItem(CartItemKey key, int qty, Money snapshot, Map<String,String> opts) {
        if (qty < 1) throw new IllegalArgumentException("qty>=1");
        this.key = key; this.quantity = qty;
        this.unitPriceSnapshot = snapshot; this.selectedOptions = opts;
    }
    Money lineTotal() { return unitPriceSnapshot.times(quantity); }
}

public class Cart {
    final String cartId; final String userId; final String currency;
    private final Map<CartItemKey, CartItem> items = new LinkedHashMap<>();
    private String coupon;
    private final PricingEngine pricing;
    private final Catalog catalog;      // live price + stock lookups

    // Idempotent on (product,variant): increment if present, else insert w/ fresh snapshot.
    public CartItem addItem(String productId, String variantId, int qty) {
        var key = new CartItemKey(productId, variantId);
        int stock = catalog.availableStock(key);
        var existing = items.get(key);
        int target = (existing == null ? 0 : existing.quantity) + qty;
        int clamped = Math.min(target, stock);
        if (clamped < 1) throw new OutOfStockException(key);
        if (existing != null) { existing.quantity = clamped; return existing; }
        var snap = catalog.currentPrice(key);
        var item = new CartItem(key, clamped, snap, Map.of());
        items.put(key, item);
        return item;
    }

    public void updateQuantity(CartItemKey key, int qty) {
        if (qty == 0) { items.remove(key); return; }
        var it = items.get(key);
        if (it == null) throw new NoSuchElementException();
        it.quantity = Math.min(qty, catalog.availableStock(key));
    }

    public void removeItem(CartItemKey key) { items.remove(key); }
    public void clear() { items.clear(); coupon = null; }
    public void applyCoupon(String code) { this.coupon = code; }

    public CartSummary getSummary() {
        Money subtotal = items.values().stream()
            .map(CartItem::lineTotal)
            .reduce(Money.zero(currency), Money::plus);
        return pricing.price(new PriceContext(items.values(), userId, coupon, subtotal));
    }

    // Merge a guest cart into this one on login.
    public void merge(Cart guest) {
        for (var gi : guest.items.values())
            addItem(gi.key.productId(), gi.key.variantId(), gi.quantity);
    }

    // Detect stock/price drift before checkout.
    public List<CartChange> revalidate() {
        var changes = new ArrayList<CartChange>();
        for (var it : items.values()) {
            var live = catalog.currentPrice(it.key);
            if (!live.equals(it.unitPriceSnapshot)) changes.add(CartChange.priceChanged(it.key, live));
            if (catalog.availableStock(it.key) < it.quantity) changes.add(CartChange.stockLow(it.key));
        }
        return changes;
    }
}
```

**Example input / output:**

```text
Upsert semantics of addItem (the key behaviour):

Start: cart empty.
addItem(p9, v-red-M, 1)      -> items = { (p9,v-red-M): qty 1, snap 1299.00 }
addItem(p9, v-red-M, 1)      -> SAME key -> increment -> qty 2  (NOT a new row)
addItem(p9, v-blue-M, 1)     -> DIFFERENT variant key -> new line item, qty 1
updateQuantity((p9,v-red-M), 0) -> qty 0 -> line removed
getSummary()                 -> subtotal from remaining lines, then pricing pipeline
                                applies coupon/tax -> grand total

revalidate() when catalog price of (p9,v-blue-M) rose 1299.00 -> 1499.00:
  returns [ priceChanged((p9,v-blue-M), 1499.00) ]  -> UI asks user to accept
```

**Data:**

```

-- One row per (cart, product, variant): the unique key enforces the core invariant -- so concurrent adds can't create duplicate line items. CREATE TABLE cart ( cart_id UUID PRIMARY KEY, user_id UUID NULL, -- NULL = guest cart session_id TEXT NULL, -- for guest identification currency CHAR(3) NOT NULL, coupon_code TEXT NULL, created_at TIMESTAMPTZ NOT NULL DEFAULT now(), updated_at TIMESTAMPTZ NOT NULL DEFAULT now() );

CREATE TABLE cart_item ( cart_item_id UUID PRIMARY KEY, cart_id UUID NOT NULL REFERENCES cart(cart_id) ON DELETE CASCADE, product_id UUID NOT NULL, variant_id UUID NOT NULL, quantity INT NOT NULL CHECK (quantity >= 1), unit_price_minor BIGINT NOT NULL, -- price snapshot at add-time currency CHAR(3) NOT NULL, selected_options JSONB NOT NULL DEFAULT '{}', added_at TIMESTAMPTZ NOT NULL DEFAULT now(), UNIQUE (cart_id, product_id, variant_id) -- the de-dup invariant );

CODE
Copy
```json
// GET /carts/{id} — read model (totals computed, not stored)
{
  "cartId": "c-123",
  "currency": "INR",
  "items": [
    { "productId": "p-9", "variantId": "v-red-M", "quantity": 2,
      "unitPrice": 129900, "lineTotal": 259800 }
  ],
  "coupon": "WELCOME10",
  "summary": { "subtotalMinor": 259800, "discountMinor": 25980,
               "taxMinor": 42058, "grandTotalMinor": 275878 }
}
```

**Flow:**

1. contains (keyed)
2. has
3. references
4. uses
5. 1
6. 1
7. *
8. *
9. Cart
10. +String cartId
11. +String userId
12. +String currency
13. +addItem(pid, vid, qty) : CartItem
14. +updateQuantity(key, qty)
15. +removeItem(key)
16. +clear()
17. +applyCoupon(code)
18. +getSummary() : CartSummary
19. +merge(other)
20. +revalidate() : List<CartChange>
21. CartItem
22. +CartItemKey key
23. +int quantity
24. +Money unitPriceSnapshot
25. +Map options
26. +lineTotal() : Money
27. Product
28. +String id
29. +String name
30. +Money basePrice
31. Variant
32. +String variantId
33. +String sku
34. +Map attrs
35. +Money price
36. +int stock
37. PricingEngine
38. +price(ctx) : CartSummary
39. yes
40. no
41. addItem product,variant,qty
42. key exists in cart?
43. increment quantity
44. snapshot current price
45. create CartItem
46. clamp to stock/limit
47. persist upsert on unique key
48. return updated line

