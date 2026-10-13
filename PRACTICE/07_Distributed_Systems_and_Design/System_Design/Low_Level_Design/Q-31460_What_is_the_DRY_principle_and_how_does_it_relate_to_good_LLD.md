---
type: "practice"
difficulty: "Basic"
---

# What is the DRY principle and how does it relate to good LLD?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Over-engineering vs Simplicity](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Define the DRY ("Don't Repeat Yourself") principle precisely, and explain why it matters for Low-Level Design (LLD) — i.e., how it shapes the classes, methods, and modules you draw on the whiteboard so the design stays maintainable.

What the interviewer is probing for (basic-concept level): Do you know the actual definition (many people mis-state it as "never copy-paste code"), can you connect it to real LLD benefits (single source of truth, fewer bugs, easier change), and — importantly — do you know its limits (over-DRYing creates the wrong coupling)? A strong fresher answer defines the term, gives a concrete before/after, and shows judgment about when NOT to apply it.

Plan to reach the answer: (1) state the precise definition, (2) explain the distinction between duplicated code and duplicated knowledge, (3) show a small concrete example of removing duplication, (4) connect each DRY benefit to an LLD quality (maintainability, correctness, extensibility), (5) cover the danger of over-applying it (wrong abstraction / accidental coupling), (6) summarize.

What DRY actually says

DRY was coined by Andy Hunt and Dave Thomas in The Pragmatic Programmer. The precise wording is:

"Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."

The keyword is knowledge, not text. DRY is about not having the same decision, rule, or fact expressed in two places — because when that fact changes, you must remember to change every copy, and sooner or later you forget one and introduce a bug. Copy-pasted code is the most obvious symptom, but DRY also covers duplicated business rules, duplicated constants, duplicated validation logic, even the same fact stated in both the schema and the code.

So the crisp definition to give: DRY means each piece of knowledge in the system lives in exactly one place; you eliminate duplication so a change is made once and takes effect everywhere.

A concrete example

Suppose an e-commerce app computes the final price in three places — the cart page, the checkout API, and the invoice PDF:

CODE
finalPrice = price * quantity * (1 - discount) * (1 + taxRate)

That pricing rule is one piece of knowledge copied three times. The day tax handling changes (say, tax applies before discount), you must edit all three — and if the invoice code is missed, customers get a wrong bill. The DRY fix is to put the rule in one method, PriceCalculator.finalPrice(...), and have all three callers use it. Now the knowledge lives in one place.

How DRY relates to good LLD

LLD is about arranging classes, methods, and responsibilities so the code is easy to understand and change. DRY is one of the core forces that pushes an LLD toward quality:

Single source of truth → correctness. A rule defined once cannot disagree with itself. In LLD terms, this usually means one class owns a responsibility (this also echoes the Single Responsibility Principle). If two classes both know how to compute tax, they can drift apart.
Cheaper, safer change (maintainability). When a requirement changes, you edit one method instead of hunting for N copies. Good LLD is judged largely by how localized a change is; DRY shrinks the "blast radius" of edits.
Fewer bugs. Every duplicate is a place a fix can be forgotten. Removing duplication removes whole classes of "we fixed it in one place but not the other" defects.
Readability and smaller surface. One well-named method (isEligibleForFreeShipping()) communicates intent better than the same 6-line condition scattered around.

In practice you apply DRY in LLD through everyday techniques: extract a method for a repeated block, extract a class/service for repeated behavior, use constants/enums instead of magic values, use inheritance or composition to share behavior, use utility/helper functions, and use generics/parameters to fold near-identical variants into one.

The critical nuance: don't over-DRY

This is what separates a rote answer from a good one. DRY is about knowledge, so two blocks of code that look the same but represent different decisions should not be merged. If you force them together, a later change to one meaning breaks the other — you created accidental coupling and "the wrong abstraction."

Classic example: validateUsername() and validatePassword() might both currently be "length between 8 and 20." They look identical, but they are different rules that happen to coincide today. Merge them into one validateField(8, 20) and the day passwords must allow 64 characters, you either break usernames or add a special case — the abstraction fights you.

The mature rule of thumb is "duplication is far cheaper than the wrong abstraction" (Sandi Metz). Prefer to wait until you see the same knowledge repeated ~three times with the same reason to change (the "Rule of Three") before extracting it. So a good LLD applies DRY to genuine shared knowledge, and tolerates surface-level similarity when the underlying rules are independent.

#### Common mistakes
/ follow-up probes
Mistake: treating DRY as "never write the same line twice." Correct framing: never duplicate the same knowledge.
Mistake: over-abstracting early, creating a tangled base class or "God util" that everything depends on. That hurts LLD more than a little duplication.

#### Follow-up
an interviewer may ask: "How does DRY relate to SRP and coupling?" — DRY pushes each responsibility into one owner (SRP), but if done to unrelated code it increases coupling. Good design balances the two.

#### Follow-up
"When would you deliberately duplicate?" — across service boundaries (two microservices shouldn't share a library just to avoid a tiny duplicate rule, as that couples their deploys), or when two look-alikes are genuinely independent.
Summary — what a strong answer emphasizes

DRY = every piece of knowledge has one authoritative home. It relates to good LLD because it gives you a single source of truth, so changes are made once, bugs from out-of-sync copies disappear, and the design stays maintainable and readable. But it is about knowledge, not textual similarity: over-applying DRY to code that merely looks alike creates the wrong abstraction and accidental coupling. The best LLD applies DRY to real shared rules (extract methods/classes, use constants), respects the Rule of Three, and knowingly tolerates small duplication when the alternative is a bad abstraction.

**Comparison:**

```
Aspect	With DRY (knowledge in one place)	With duplication
Cost of a rule change	1 edit	N edits, must find all copies
Risk of inconsistency	Impossible (one source of truth)	High — forget a copy → bug
Readability	Named method conveys intent	Same logic re-read everywhere
When it backfires	If you merge unrelated look-alike code → wrong abstraction / accidental coupling	Small independent duplicates are sometimes fine
```

**Code:**

```java
// BEFORE — the pricing rule (one piece of knowledge) is duplicated in 3 callers
class CartPage {
    double show(double price, int qty, double discount, double tax) {
        return price * qty * (1 - discount) * (1 + tax);   // copy 1
    }
}
class CheckoutApi {
    double charge(double price, int qty, double discount, double tax) {
        return price * qty * (1 - discount) * (1 + tax);   // copy 2
    }
}
// ...InvoicePdf has copy 3 — change the rule once and you must fix all three.

// AFTER — single source of truth: the knowledge lives in exactly one place
class PriceCalculator {
    static double finalPrice(double price, int qty, double discount, double tax) {
        return price * qty * (1 - discount) * (1 + tax);
    }
}
class CartPage2    { double show(double p,int q,double d,double t){ return PriceCalculator.finalPrice(p,q,d,t);} }
class CheckoutApi2 { double charge(double p,int q,double d,double t){ return PriceCalculator.finalPrice(p,q,d,t);} }
// Now a tax-rule change is edited in ONE method and every caller is correct.
```

**Example input / output:**

```text
Worked scenario (why the single source of truth matters):

Requirement change: "apply tax to the pre-discount price."
- DRY design: edit `PriceCalculator.finalPrice` once → cart, checkout, and invoice all become consistent. 1 edit, 0 missed spots.
- Duplicated design: must edit 3 copies. If the invoice copy is missed → customer is charged X on checkout but billed Y on the invoice → a real, hard-to-notice bug.
```

**Flow:**

1. Yes
2. No, just looks similar
3. Spot repeated code
4. Same *knowledge*same reason to change?
5. Extract to one placemethod / class / constant
6. Single source of truthmaintainable LLD
7. Leave duplicated
8. Avoid wrong abstraction& accidental coupling

