---
type: "practice"
difficulty: "Deep"
---

# What is the Template Method pattern and how does it use inheritance?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composition vs Inheritance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Composition_vs_Inheritance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is being asked: Explain the Template Method design pattern and, specifically, how it leverages inheritance. This is a "deep understanding" LLD question from strong product companies (Amazon, Microsoft, Atlassian, Uber, Razorpay, Postman, Flipkart, Zoho). They don't want a textbook definition — they want you to show you understand why the pattern exists, the mechanics of the inversion it creates, the trade-offs, and when it hurts.

What the interviewer is probing for: Do you understand the "skeleton in the base class, steps in subclasses" inversion? Do you know the difference between the fixed template method and the overridable primitive operations? Can you articulate the Hollywood Principle ("don't call us, we'll call you"), hooks, and the tension between inheritance and composition (Strategy)?

#### Assumptions
Java-flavored answer (typical for LLD rounds). I'll ground it in a concrete, realistic example — a data-processing / report pipeline — rather than an abstract "Game" toy.

Plan: define the pattern → dissect its parts → show how inheritance provides the mechanism → concrete code → trade-offs vs Strategy → failure modes and the "final" discipline → summary.

What the Template Method pattern is

Template Method is a behavioral pattern that defines the skeleton of an algorithm in a base-class method — the template method — while deferring some individual steps to subclasses. The overall sequence of steps is fixed and lives in one place; the variable steps are declared as abstract (or overridable) methods that concrete subclasses fill in. So the algorithm's structure is invariant, but specific behaviors vary by subclass.

Concretely: a process() method in the base class calls readData(), transform(), then writeData() in a fixed order. process() itself is marked final so no subclass can reorder or break the algorithm. readData/transform/writeData are abstract; a CsvProcessor and a JsonProcessor each override them.

How it uses inheritance — the key mechanism

This is the crux of the question. Template Method relies on inheritance + polymorphism (overriding) as its engine:

The base class owns control flow. The template method is a concrete, final method in the abstract base class. It sequences the steps and enforces the invariant algorithm. This is the "template" — the reusable, don't-repeat-yourself part.

Primitive operations are abstract. The steps that vary are declared abstract. The base class calls them even though it doesn't implement them. This is only possible because of dynamic dispatch — when the base's process() calls this.transform(), the JVM resolves transform() to the actual subclass's override at runtime.

Inversion of control (the Hollywood Principle). Normally your code calls library code. Here it flips: the base class (the framework) calls your subclass's methods. "Don't call us, we'll call you." The subclass plugs behavior in; it does not drive the flow. This is exactly how framework hook methods (doGet/doPost in HttpServlet, JUnit's setUp/tearDown, Spring's AbstractController) work.

Hooks. Besides mandatory abstract steps, the base can provide hook methods — non-abstract methods with an empty or default body that subclasses may override to customize optional points (e.g., boolean shouldValidate() defaulting to true, or an empty afterProcess()). Hooks let subclasses opt into extension without being forced to.

So inheritance is not incidental here — it is the delivery mechanism. The base class defines the algorithm and the extension points; subclasses supply the variable behavior through overriding. Without inheritance/virtual dispatch there is no Template Method.

The important discipline: make the template final

A subtle but interview-worthy point: the template method should be final (or otherwise sealed). If a subclass could override process() itself, it could reorder or skip steps and violate the very invariant the pattern protects. Meanwhile, the primitive operations should be abstract (must implement) or protected hooks (may implement), and ideally not part of the public API — clients call the template, not the steps.

#### Trade-offs
— Template Method vs Strategy

The most common follow-up: "Why not Strategy?" Both let behavior vary, but through different axes:

Template Method = inheritance (compile-time, static). One class hierarchy; variation is baked in at subclass definition. Simple, but you get all the downsides of inheritance: tight coupling to the base, no runtime swapping, single-inheritance limits combining variations, and the fragile-base-class problem (a base change can silently break subclasses).
Strategy = composition (runtime, dynamic). The algorithm's variable step is a separate object injected in. You can swap it at runtime and combine strategies freely; favored by "prefer composition over inheritance." Cost: more objects/wiring.

Rule of thumb: use Template Method when the algorithm's skeleton is truly stable and variations are few and known at design time; use Strategy when you need runtime flexibility or many orthogonal variations. They're not mutually exclusive — a Template Method's individual step can itself be a Strategy.

Failure modes to mention
Overuse / deep hierarchies: every new variation is a new subclass; combinatorial explosion when two independent axes vary (that's a Strategy/Bridge signal).
Fragile base class: changing the template or a hook can break every subclass. Keep the base contract stable and documented.
Leaky primitives: if primitive operations are public, clients call them out of order. Keep them protected/abstract.
Calling overridable methods from a constructor (in Java) — the subclass field isn't initialized yet; a classic bug when the template runs too early.
Real-world sightings

java.io.InputStream.read(byte[]) (template) calling read() (primitive); HttpServlet.service() dispatching to doGet/doPost; JUnit's test lifecycle; AbstractList / AbstractMap; Spring's JdbcTemplate.execute(...) (though it also composes callbacks). Naming them shows you recognize the pattern in the wild.

Summary — what a strong answer emphasizes

Template Method puts the invariant algorithm skeleton in a final base method and defers variable steps to abstract/hook methods overridden by subclasses. It uses inheritance + dynamic dispatch so the base class calls into the subclass — an inversion of control (Hollywood Principle). Emphasize: final template + abstract/hook primitives, hooks for optional customization, and the deliberate trade-off vs Strategy (static inheritance vs runtime composition). Mention the fragile-base-class risk and the constructor-dispatch pitfall to show senior depth.

**Comparison:**

```
Dimension	Template Method	Strategy
Reuse mechanism	Inheritance (subclass overrides steps)	Composition (inject algorithm object)
Binding of variation	Compile time / static	Runtime / dynamic (swappable)
What varies	Individual steps of a fixed skeleton	The whole algorithm/step object
Coupling	Tight to base class	Loose (interface only)
Combine many variations	Hard (single inheritance, subclass blow-up)	Easy (mix strategies)
Best when	Skeleton stable, few known variants	Need runtime flexibility / many variants
Risk	Fragile base class, ctor-dispatch bug	More objects/wiring
Member kind	Modifier	Overridable?	Role
Template method	final	No (by design)	Fixed algorithm skeleton
Primitive operation	abstract	Must implement	Required variable step
Hook (default body)	protected	May override	Optional customization point
```

**Code:**

```java
// Abstract base class: owns the ALGORITHM SKELETON (the template)
abstract class DataProcessor {

    // The TEMPLATE METHOD — final so subclasses can't reorder/break the algorithm
    public final void process() {
        var raw = readData();          // primitive (subclass supplies)
        var clean = transform(raw);    // primitive (subclass supplies)
        if (shouldValidate()) {        // HOOK (optional override)
            validate(clean);
        }
        writeData(clean);              // primitive (subclass supplies)
        afterProcess();                // HOOK (empty default)
    }

    // Primitive operations — MUST be implemented by subclasses
    protected abstract String readData();
    protected abstract String transform(String raw);
    protected abstract void writeData(String data);

    // Hook with a default — subclass MAY override
    protected boolean shouldValidate() { return true; }
    protected void validate(String data) {
        if (data == null || data.isBlank())
            throw new IllegalStateException("empty data");
    }
    // Hook with empty body — subclass MAY override
    protected void afterProcess() { /* no-op by default */ }
}

// Concrete subclass fills in the variable steps
class CsvProcessor extends DataProcessor {
    protected String readData() { return "a,b,c"; }
    protected String transform(String raw) { return raw.toUpperCase(); }
    protected void writeData(String data) { System.out.println("CSV out: " + data); }
    protected void afterProcess() { System.out.println("CSV done"); }
}

class JsonProcessor extends DataProcessor {
    protected String readData() { return "{\"k\":\"v\"}"; }
    protected String transform(String raw) { return raw.replace("\"", "'"); }
    protected void writeData(String data) { System.out.println("JSON out: " + data); }
    protected boolean shouldValidate() { return false; } // opt out of validation
}

public class Demo {
    public static void main(String[] args) {
        DataProcessor p = new CsvProcessor();
        p.process();                    // base calls INTO subclass steps
        new JsonProcessor().process();
    }
}
```

**Example input / output:**

```text
// new CsvProcessor().process():
//   readData()="a,b,c" -> transform="A,B,C" -> shouldValidate()=true -> validate ok -> write -> afterProcess
CSV out: A,B,C
CSV done

// new JsonProcessor().process():
//   readData={"k":"v"} -> transform={'k':'v'} -> shouldValidate()=false (validation skipped) -> write -> default afterProcess (no-op)
JSON out: {'k':'v'}
```

**Flow:**

1. CsvProcessor (subclass)
2. DataProcessor (base)
3. Client
4. CsvProcessor (subclass)
5. DataProcessor (base)
6. Client
7. Hollywood Principle:
8. "Don't call us, we'll call you"
9. process() [final template]
10. readData() %% base calls INTO subclass (inversion of control)
11. raw
12. transform(raw)
13. clean
14. shouldValidate()? validate() [hook + default]
15. writeData(clean)
16. afterProcess() [hook]

