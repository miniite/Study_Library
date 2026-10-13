---
type: "practice"
difficulty: "Deep"
---

# Where would you put validation logic in a layered design and why not in the entity?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Extensible Rule Engines](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Extensible_Rule_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The question has two parts, and the interviewer wants both. First: where does validation live in a layered (controller → service → domain → repository) design. Second, and this is the real probe: why is "just put it in the entity" not a complete answer? At "deep understanding" level, a shallow reply ("validate in the service") fails. The interviewer is testing whether you understand that validation is not one thing — there are several kinds of validation, each with a natural home — and whether you can reason about coupling, reuse, and where truth about correctness should ultimately be enforced.

Clarifying assumptions I would state up front: we are talking about a typical layered/hexagonal backend (say a Razorpay payment API or an Amazon order service); the "entity" means a rich domain object, not an anemic data bag; and "validation" spans syntax, semantics, and business invariants. With that framing, here is the full picture.

The key insight: there are three different validations, not one

The trap in "put it in the entity" is that it treats validation as monolithic. In practice there are three distinct categories, and they belong in three different places:

Input/syntactic validation — is the request even well-formed? Is email a string, is amount a number ≥ 0, is quantity an integer, are required fields present, is the JSON shape right? This is about the shape of the incoming DTO, and it depends on the transport (HTTP), not on the business domain.

Application/orchestration validation — cross-cutting, use-case-specific rules that need collaborators: "does this SKU exist?", "is this coupon still active?", "does the buyer have permission to place this order?", "is the account not frozen?". These require repositories, other services, or external calls.

Domain invariants — rules that must ALWAYS hold for the object to be valid regardless of use case: an Order total equals the sum of its line items; a Money amount is never negative; a PaymentCapture cannot exceed the authorized amount; an Account balance never goes below its overdraft limit.

Once you split validation this way, the placement almost falls out on its own:

Syntactic input validation → at the edge (controller / DTO layer). Use annotations/schemas (Bean Validation @NotNull/@Positive, Zod, class-validator, FluentValidation) on the request DTO. Reject a 400 before any business logic runs. This keeps malformed junk out of the core and gives clean, uniform 400 responses.

Application/orchestration validation → in the service (application) layer. These rules need repositories and other services, which the pure domain must not depend on. The service is the only layer that can legitimately load the SKU, check the coupon, verify permissions, and then call the domain.

Domain invariants → inside the domain/entity itself. This is the crucial nuance: the entity IS the right place for the rules that define what it means for that object to be valid, and it enforces them by rejecting construction/mutation (constructor guards, factory methods, and methods that never let the object enter a bad state).

So the honest answer is not "not in the entity." It is: invariants belong in the entity; the other two kinds do not. The interesting part is why the other two don't.

Why input and orchestration validation should NOT live in the entity

It couples the domain to the transport. If the entity validates HTTP-shaped input (string lengths, request-field presence, formats tied to a JSON payload), the domain now knows about the API. Tomorrow the same entity is used from a batch import, a Kafka consumer, or a gRPC call with a different shape — and the entity's input rules are wrong or in the way. Input validation is transport-specific; the domain must be transport-agnostic.

It forces the domain to depend on infrastructure. "Does the SKU exist?" needs a repository. "Is the user allowed?" needs auth context. If the entity did this, the domain would have to reach out to databases and services — inverting the dependency direction (domain should have zero outward dependencies) and making entities un-unit-testable without mocks and a container.

Entities are created in valid states — they shouldn't be born invalid to be checked later. A rich entity should be impossible to construct in an invalid state (fail fast in the constructor/factory). If you push "validate the raw request" into the entity, you either can't build the object until it's valid (so the request-shape checks happen before construction anyway — i.e., not in the entity), or you build a half-valid object and call .validate() on it — which is the anemic anti-pattern and leaves a window where a bad object exists.

Reuse and single-responsibility. Input rules change with the API contract; invariants change with the business. Mixing them means an API tweak forces edits to the domain and risks breaking core rules. Separating them lets each evolve independently.

Putting it together — defense in depth, not duplication

A common misread is "so I validate the same thing three times?" No. Each layer validates different things:

Controller: shape/format (fast fail, cheap).
Service: context/authorization/existence (needs collaborators).
Entity: invariants (last line of defense, protects the object no matter who calls it).

The entity's invariant check is the authoritative, non-bypassable guard. Even if a future caller skips the service, or a bug lets bad data through, the entity still refuses to enter a bad state. That is why invariants live there and nowhere else — they must hold universally.

What a strong answer emphasizes
Validation is plural: syntactic (edge/DTO), semantic/orchestration (service), invariant (domain).
The entity owns invariants and enforces them by construction (fail-fast constructors, factory methods, guarded mutators) — an entity should be impossible to hold in an invalid state.
It does NOT own input/transport validation (couples to transport) or existence/authorization checks (would force infra dependencies and invert the dependency rule).
This is defense in depth, not duplication — different concerns at different layers.

#### Common mistakes
/ follow-up probes
Anemic domain: putting all logic in services and using entities as data bags — then invariants aren't guaranteed and can be bypassed. Interviewers push here: "if your entity has no validation, what guarantees the total is right?"
Fat entity: pulling repositories into the entity to do existence checks — breaks the dependency rule. Follow-up: "how would you unit-test that entity without a DB?"
Double 400/500 confusion: input errors should be 400 at the edge; a violated invariant reaching the entity usually signals a bug (should have been caught upstream) and is a 500 or a deliberate domain exception mapped to a 422. A good candidate distinguishes expected user errors (validate early, return 4xx) from impossible states (invariants, defensive).

#### Follow-up
they love: "Where do you validate that a coupon is still valid — entity or service?" Correct: service (needs to load the coupon and check time), while "a discount can't exceed the order total" is an invariant on the entity.

**Comparison:**

```
Validation kind	Example	Home layer	Why not in the entity
Syntactic / input shape	required field, amount is a number, email format	Controller / DTO (Bean Validation, Zod)	Ties domain to HTTP transport; wrong for batch/Kafka reuse
Orchestration / semantic	SKU exists, coupon active, user authorized	Service (application) layer	Needs repositories/other services → would invert the dependency rule and make entities un-unit-testable
Domain invariant	total = Σ line items; discount ≤ subtotal; balance ≥ overdraft	Entity (constructor/factory/guarded mutators)	This IS the entity's job — enforced by construction
```

**Code:**

```java
// ---- Edge: DTO carries transport/syntactic validation (Bean Validation) ----
public record CreateOrderRequest(
    @NotBlank String customerId,
    @NotEmpty List<@Valid LineDto> lines,
    @Positive long couponAmountPaise      // shape only, not business truth
) {}

@RestController
class OrderController {
  @PostMapping("/orders")
  ResponseEntity<?> create(@Valid @RequestBody CreateOrderRequest req) { // 400 if malformed
    return ResponseEntity.ok(orderService.place(req));
  }
}

// ---- Application/service: orchestration validation needing collaborators ----
@Service
class OrderService {
  private final SkuRepository skus;
  private final CouponRepository coupons;

  Order place(CreateOrderRequest req) {
    // existence / authorization / cross-aggregate rules live HERE, not in the entity
    var items = req.lines().stream()
        .map(l -> skus.findById(l.sku())
            .orElseThrow(() -> new NotFoundException("SKU " + l.sku()))
            .toLineItem(l.qty()))
        .toList();
    Coupon coupon = coupons.findActive(req.customerId())    // needs a repo -> not in entity
        .orElse(Coupon.none());
    // domain invariants enforced by construction, below
    return Order.place(req.customerId(), items, coupon);
  }
}

// ---- Domain entity: invariants only, enforced by construction (fail fast) ----
class Order {
  private Order(String customerId, List<LineItem> items, Money discount) {
    if (items.isEmpty())            throw new DomainException("order needs >=1 line");
    Money subtotal = items.stream().map(LineItem::total).reduce(Money.ZERO, Money::plus);
    if (discount.gt(subtotal))      throw new DomainException("discount exceeds subtotal"); // invariant
    // object cannot exist in an invalid state past this point
    this.customerId = customerId; this.items = List.copyOf(items); this.total = subtotal.minus(discount);
  }
  static Order place(String customerId, List<LineItem> items, Coupon c) {
    return new Order(customerId, items, c.amountFor(items)); // factory = only valid construction path
  }
}
```

**Example input / output:**

```

Request (malformed shape) → rejected at the EDGE, entity never runs:

CODE
Copy
POST /orders   { "customerId": "", "lines": [] }
→ 400 Bad Request
  { "errors": ["customerId: must not be blank", "lines: must not be empty"] }

Request (valid shape, unknown SKU) → rejected by the SERVICE (needs repository):

CODE
Copy
POST /orders   { "customerId":"c1", "lines":[{"sku":"X9","qty":2}], "couponAmountPaise":100 }
→ 404 Not Found   { "error": "SKU X9" }

Request (valid shape + real SKUs, discount > subtotal) → INVARIANT enforced by the entity:

CODE
Copy
subtotal = 150 paise, requested discount = 100000 paise
→ DomainException: "discount exceeds subtotal"  (mapped to 422)
```

**Flow:**

1. no
2. yes
3. no
4. yes
5. no
6. yes
7. HTTP Request
8. Controller/DTO: shape valid?
9. 400 Bad Request
10. Service: exists? authorized?coupon active?
11. 404 / 403 / 409
12. Entity constructor/factory:invariants hold?
13. DomainException -> 422 /bug=500
14. Valid object persisted viaRepository

