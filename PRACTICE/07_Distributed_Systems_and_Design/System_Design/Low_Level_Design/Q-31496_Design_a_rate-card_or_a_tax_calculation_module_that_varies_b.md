---
type: "practice"
difficulty: "Deep"
---

# Design a rate-card or a tax calculation module that varies by region. Which pattern?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Open-closed Extensibility](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Open-closed_Extensibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: "Design a rate-card / tax calculation module whose rules vary by region — which pattern?" The interviewer wants me to model a piece of business logic that is (a) heavily variant (India GST vs. US sales tax vs. EU VAT vs. UAE VAT), (b) changes over time (tax rates are amended by regulation), and (c) must be auditable (a fintech/e-commerce company must be able to prove why an order was charged ₹18 tax). The "which pattern?" hint is the tell: they want me to name and justify a pattern, and to show I won't reach for a giant if region == "IN" … else if … switch.

What they're probing for: Do I know the Strategy pattern and when to combine it with a Factory/Registry for runtime selection? Do I separate policy (which rule applies) from mechanism (how the math is done)? Do I treat rates as data, not code so a rate change doesn't need a deploy? Do I think about composition (a single invoice can have a base price + a region tax + a possible surcharge/cess) and about time-versioning and audit?

Clarifying questions I'd ask:

Is this a tax module, a rate-card/pricing module, or both? (They're structurally the same — a chain of context-dependent calculators — so I'll design one engine that serves both.)
Region granularity? Country only, or country + state (US has state + county), or country + product-category (India taxes goods vs. digital services differently)?
Do rates change often, and do we need to compute historical tax (a refund issued today for an order placed last quarter must use last quarter's rate)?
Composability: can multiple taxes stack (e.g., India CGST + SGST, or a state tax + a city surcharge)?

Plan: Model each region's calculation as a Strategy; select the strategy at runtime with a Registry/Factory keyed by region (+ category); make the actual rates externalized data (DB/config) with effective-date versioning; allow composition of multiple tax components; return a rich, itemized result for audit. I'll close with the pattern comparison and pitfalls.

From first principles to the design
Step 1 — Name the axis of variation

The one thing that changes is the calculation itself, chosen by region. Whenever "the algorithm varies by a key selected at runtime," the textbook answer is the Strategy pattern: define a common interface, put each variant behind it, and pick the right one at runtime. This is the core answer to "which pattern."

But region-by-region behaviour is only half the problem. Two things kill a naïve Strategy-only design:

Rates are volatile. A class IndiaTaxStrategy { rate = 0.18 } means every GST amendment is a code change + deploy + release review. Unacceptable for a Razorpay/ClearTax. So the structure (how to compute) is code (Strategy), but the rate values are data loaded from a store.
You must select the strategy. Something has to map "IN" → the right strategy. Hard-coding that mapping in a switch reintroduces the smell. So I add a Registry/Factory that resolves region → strategy, and can even be populated dynamically.
Step 2 — The interface (mechanism)
CODE
interface TaxCalculator {
    TaxResult calculate(TaxContext ctx);
}

TaxContext carries everything a rule could need: amount, region (country/state/city), productCategory, customerType (B2B vs B2C — affects reverse-charge/VAT), date (for historical calc), and currency. TaxResult is not a bare number — it's an itemized breakdown ([{name:"CGST", rate:9%, amount:₹9}, {name:"SGST", rate:9%, amount:₹9}], total ₹18). Returning the breakdown is what makes it auditable and invoice-ready.

Step 3 — Rates as versioned data

A RateRepository returns the applicable rate rows for (region, category, date). Each row has effectiveFrom/effectiveTo, so historical refunds compute correctly. The strategy asks the repo for rates and does the arithmetic; it holds no literals. Changing a GST slab = an INSERT with a new effectiveFrom, no deploy.

Step 4 — Composition for stacked taxes

India charges CGST and SGST (intra-state) or IGST (inter-state). US may charge state + county. So a region's strategy is often a Composite: it iterates the rate components for that region and sums them, each producing its own line item. This is the Strategy + Composite combination.

Step 5 — Selection (Registry/Factory)
CODE
class TaxCalculatorRegistry {
    register(region, factory);        // extend without editing existing code (OCP)
    TaxCalculator resolve(region);    // falls back to a DefaultCalculator / throws for unknown
}

New country ⇒ register a new strategy; no existing file is touched — that's the Open/Closed Principle in action, and it's the concrete payoff over a switch statement.

Step 6 — Optional: rule-as-data / spec engine

For very rule-heavy domains (ClearTax-style), some teams push even the logic into data — a small rules engine where conditions like "if category=digital AND region=EU AND customer=B2C then apply destination VAT" are table-driven. That's a valid escalation, but it's harder to test and debug, so I'd only reach for it when the rate structure (not just the numbers) changes constantly.

Which pattern — and the honest trade-offs
Strategy — the primary answer; one interface, per-region implementations, runtime selection. Clean, testable, OCP-friendly.
Factory / Registry — to select the strategy without a switch; makes it plug-in extensible.
Composite — to stack multiple tax components into itemized line items.
Chain of Responsibility — a viable alternative when taxes are sequential/conditional ("apply base tax, then a surcharge only if amount > ₹X"); each handler decides to act and pass on. I'd use it over Composite when order and short-circuiting matter.
Rules engine / interpreter — for the highest-variance case, logic itself as data.

The pattern I'd lead with is Strategy + Registry, with rates externalized as versioned data and results itemized. That directly satisfies variance-by-region, no-deploy rate changes, historical accuracy, and audit — the four things a fintech/e-commerce interviewer is actually testing.

#### Common mistakes
/ follow-up probes
Baking rates into code — the #1 miss; rates must be data.
Returning a single number — you lose auditability and can't render an invoice; return line items.
Ignoring time — refunds/amendments need effective-dated rates.
A hidden switch in the factory — acceptable at the edge (config-driven registration is better), but don't let region logic leak back into the calculators.

#### Follow-ups
they'll push on: rounding rules (per-line vs. per-invoice, banker's rounding for tax), currency/precision (use integer minor-units or BigDecimal, never float), reverse-charge/B2B, tax-exempt entities, and idempotent recomputation.

A strong answer emphasizes: Strategy for the varying algorithm, Registry/Factory for selection (OCP), rates as versioned data not code, Composite/Chain for stacking, and a rich itemized, effective-dated result for audit and invoicing.

**Comparison:**

```
Pattern	When to use here	Pros	Cons
Strategy	Algorithm varies by region (primary)	Clean, testable, OCP	Need a selector
Factory/Registry	Map region → strategy at runtime	Plug-in extensible, no switch	Extra indirection
Composite	Stack CGST+SGST / state+county	Itemized lines, reusable	Sum/rounding must be explicit
Chain of Responsibility	Sequential/conditional surcharges	Ordered, short-circuits	Harder to see full picture
Rules engine / rules-as-data	Logic itself changes constantly	No deploy for rule change	Harder to test/debug
❌ Giant switch/if-else	—	Fast to write	Violates OCP, unmaintainable, untestable
Concern	Design decision
Rate changes	Externalize to tax_rate table (data, not code)
Historical/refund accuracy	effective_from/effective_to versioning
Audit / invoicing	Return itemized TaxResult, not a scalar
Money precision	BigDecimal / integer minor-units; HALF_EVEN; never float
Adding a country	registry.register(...) — no existing code edited
```

**Code:**

```java
// ---- Contracts ----
record Region(String country, String state, String city) {}

record TaxContext(BigDecimal amount, Region region, String category,
                  CustomerType customerType, LocalDate date, Currency currency) {}

record TaxLine(String name, BigDecimal rate, BigDecimal amount) {}

record TaxResult(BigDecimal taxableBase, List<TaxLine> lines, BigDecimal totalTax) {
    static TaxResult of(BigDecimal base, List<TaxLine> lines) {
        BigDecimal total = lines.stream().map(TaxLine::amount)
                                .reduce(BigDecimal.ZERO, BigDecimal::add);
        return new TaxResult(base, lines, total);
    }
}

interface TaxCalculator { TaxResult calculate(TaxContext ctx); }

// ---- Rates as versioned DATA (not code) ----
record RateRow(String component, BigDecimal rate,        // 0.09 = 9%
               LocalDate effectiveFrom, LocalDate effectiveTo) {}

interface RateRepository {
    List<RateRow> ratesFor(Region region, String category, LocalDate on);
}

// ---- A data-driven strategy that COMPOSES stacked components ----
class CompositeTaxCalculator implements TaxCalculator {
    private final RateRepository rates;
    CompositeTaxCalculator(RateRepository rates) { this.rates = rates; }

    public TaxResult calculate(TaxContext ctx) {
        var rows = rates.ratesFor(ctx.region(), ctx.category(), ctx.date());
        var lines = rows.stream().map(r -> new TaxLine(
                r.component(), r.rate(),
                ctx.amount().multiply(r.rate())
                            .setScale(2, RoundingMode.HALF_EVEN)))   // per-line rounding
            .toList();
        return TaxResult.of(ctx.amount(), lines);
    }
}

// A truly bespoke region (e.g. reverse-charge for EU B2B) gets its own strategy:
class EuVatCalculator implements TaxCalculator {
    private final RateRepository rates;
    EuVatCalculator(RateRepository rates) { this.rates = rates; }
    public TaxResult calculate(TaxContext ctx) {
        if (ctx.customerType() == CustomerType.B2B)          // reverse charge: buyer accounts VAT
            return TaxResult.of(ctx.amount(), List.of());
        return new CompositeTaxCalculator(rates).calculate(ctx);
    }
}

// ---- Registry/Factory: selection without a switch, OCP-friendly ----
class TaxCalculatorRegistry {
    private final Map<String, TaxCalculator> byCountry = new ConcurrentHashMap<>();
    private final TaxCalculator fallback;
    TaxCalculatorRegistry(TaxCalculator fallback) { this.fallback = fallback; }

    void register(String country, TaxCalculator c) { byCountry.put(country, c); }
    TaxCalculator resolve(Region r) {
        return byCountry.getOrDefault(r.country(), fallback);
    }
}

// ---- Wiring (add a country = one register() call, no existing code edited) ----
var repo = new DbRateRepository(dataSource);
var registry = new TaxCalculatorRegistry(new CompositeTaxCalculator(repo)); // default
registry.register("IN", new CompositeTaxCalculator(repo)); // CGST+SGST from data
registry.register("EU", new EuVatCalculator(repo));        // bespoke rule

// ---- Use ----
var ctx = new TaxContext(new BigDecimal("1000.00"),
        new Region("IN", "KA", "Bengaluru"), "GOODS",
        CustomerType.B2C, LocalDate.now(), Currency.getInstance("INR"));
TaxResult result = registry.resolve(ctx.region()).calculate(ctx);
```

**Example input / output:**

```text
INPUT  (India, intra-state, ₹1000 goods, B2C)
  amount=1000.00, region=IN/KA, category=GOODS, date=2026-07-07

TRACE
  registry.resolve("IN") -> CompositeTaxCalculator
  repo.ratesFor(IN, GOODS, 2026-07-07) -> [CGST 9%, SGST 9%]
  line CGST = 1000.00 * 0.09 = 90.00
  line SGST = 1000.00 * 0.09 = 90.00
  totalTax  = 180.00

OUTPUT (itemized — invoice/audit ready)
  {
    "taxableBase": "1000.00",
    "lines": [
      { "name": "CGST", "rate": "0.09", "amount": "90.00" },
      { "name": "SGST", "rate": "0.09", "amount": "90.00" }
    ],
    "totalTax": "180.00"
  }

INPUT  (EU, B2B digital) -> EuVatCalculator, reverse charge
OUTPUT { "taxableBase": "…", "lines": [], "totalTax": "0.00" }  // buyer self-accounts VAT
```

**Data:**

```

-- Rates live in a table, effective-dated for historical/refund correctness. CREATE TABLE tax_rate ( id BIGSERIAL PRIMARY KEY, country VARCHAR(2) NOT NULL, state VARCHAR(8), -- NULL = country-wide category VARCHAR(32) NOT NULL, -- GOODS / DIGITAL / SERVICE component VARCHAR(16) NOT NULL, -- CGST / SGST / IGST / VAT / STATE_TAX rate NUMERIC(6,4) NOT NULL, -- 0.0900 = 9% effective_from DATE NOT NULL, effective_to DATE -- NULL = still active );

-- India intra-state GOODS @ 18% split into CGST 9% + SGST 9% INSERT INTO tax_rate(country,state,category,component,rate,effective_from) VALUES ('IN', NULL, 'GOODS', 'CGST', 0.0900, '2024-04-01'), ('IN', NULL, 'GOODS', 'SGST', 0.0900, '2024-04-01'); -- A slab change is just a new row + closing the old one — NO deploy.

CODE
Copy
```json
// TaxContext payload (API request into the module)
{
  "amount": "1000.00",
  "region": { "country": "IN", "state": "KA", "city": "Bengaluru" },
  "category": "GOODS",
  "customerType": "B2C",
  "date": "2026-07-07",
  "currency": "INR"
}
```

**Flow:**

1. IN
2. EU
3. unknown
4. TaxContext: amount, region,category, date, customerType
5. TaxCalculatorRegistry.resolveregion.country
6. CompositeTaxCalculator
7. EuVatCalculator
8. Default/Fallback Calculator
9. RateRepository.ratesForregion, category, date
10. (tax_rate tableeffective-dated rows)
11. compose line items + roundHALF_EVEN
12. TaxResult: taxableBase, lines[],totalTax
13. Invoice / Audit log

