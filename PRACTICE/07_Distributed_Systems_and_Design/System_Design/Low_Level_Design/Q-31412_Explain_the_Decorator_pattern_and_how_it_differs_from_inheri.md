---
type: "practice"
difficulty: "Deep"
---

# Explain the Decorator pattern and how it differs from inheritance.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composition vs Inheritance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Composition_vs_Inheritance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to (a) explain the Decorator structural design pattern and (b) contrast it with inheritance. The interviewer is probing whether I understand composition over inheritance, the open/closed principle, and when to reach for runtime-attachable behavior instead of a class hierarchy. At this "deep understanding" level, a strong answer must go beyond a textbook definition: I should show the intent, the participants, a real code example, the concrete failure mode of inheritance that Decorator solves (the "class explosion" problem), and the trade-offs/pitfalls.

Clarifying assumptions: I'll assume an OO language (Java/TypeScript-style), and I'll ground it in a familiar domain — an e-commerce/beverage pricing example and a Java I/O streams example — because those are the canonical, interviewer-recognized cases.

Plan: Define intent → participants → why inheritance breaks down → Decorator code → the differences table → trade-offs and mistakes.

What the Decorator pattern is

Intent: Attach additional responsibilities to an object dynamically at runtime. Decorators provide a flexible alternative to subclassing for extending functionality. The key idea: a decorator wraps an object of the same interface, forwards calls to the wrapped object, and adds behavior before/after that call. Because a decorator is-a Component (implements the same interface) and has-a Component (holds a reference to the wrapped one), you can stack decorators arbitrarily, and clients can't tell a decorated object from an undecorated one.

Participants:

Component — the common interface (e.g., Coffee with cost() and description()).
ConcreteComponent — the base object being decorated (e.g., Espresso).
Decorator — an abstract wrapper that implements Component and holds a Component reference, delegating to it.
ConcreteDecorator — adds the actual extra behavior (e.g., MilkDecorator, SugarDecorator).
Why plain inheritance is not enough — the problem Decorator solves

Suppose a coffee shop sells Espresso, and you can add Milk, Sugar, Whip, Caramel, etc. If you model every combination with a subclass, you need EspressoWithMilk, EspressoWithMilkAndSugar, EspressoWithWhipAndCaramel… For n independent add-ons you approach 2^n classes — a combinatorial class explosion. Inheritance is decided at compile time and is static: an object's type is fixed when it's created. You cannot, at runtime, decide "this particular cup also gets extra sugar" without either a new subclass or a pile of boolean flags inside one bloated class.

Decorator fixes this by making each add-on a small, independent wrapper you can combine freely at runtime: new WhipDecorator(new MilkDecorator(new Espresso())). Each add-on is written once, order can vary, and adding a new topping is a new class that doesn't touch existing code — satisfying the Open/Closed Principle (open for extension, closed for modification).

How it differs from inheritance

They are two ways to reuse/extend behavior, but along different axes:

Binding time: Inheritance is static/compile-time — the subclass relationship is fixed. Decorator is dynamic/runtime — you choose which behaviors to layer as you build the object.
Mechanism: Inheritance uses an is-a relationship (subclass reuses/overrides parent). Decorator uses composition/delegation — an is-a (shares the interface) plus a has-a (holds and forwards to a wrapped instance).
Combinations: Inheritance forces one class per combination (exponential). Decorator uses n linear decorators that compose (O(n) classes for n features).
Coupling: Subclasses are tightly coupled to the parent's implementation (fragile base-class problem). Decorators only depend on the shared interface.
Per-instance vs per-class: Inheritance changes behavior for the whole class. A decorator changes behavior for a single object instance — two Espresso objects can have different decorations.

A useful mental model: inheritance answers "what a thing is," decoration answers "what a thing currently does."

Real-world usages
Java I/O: new BufferedReader(new InputStreamReader(new FileInputStream(f))). BufferedReader and InputStreamReader are decorators over the base stream, layering buffering and charset decoding.
Web middleware: Express/ASP.NET middleware pipelines are decorators over the request handler (auth, logging, compression each wrap the next).
UI toolkits: scrollbars/borders decorating a base widget.
Pricing/notifications: adding tax, discount, or gift-wrap to an order; layering SMS + email + push on a notifier.

#### Trade-offs
and common mistakes

#### Pros
runtime flexibility, single-responsibility per decorator, avoids class explosion, honors OCP. Cons: many small objects and deep wrapping can be hard to debug (a stack trace of 5 wrappers); the order of decorators can matter (discount-then-tax ≠ tax-then-discount); a decorator that needs to know the concrete type (peeking through the wrappers) is a smell. Common mistakes: confusing Decorator with Proxy (Proxy controls access and usually creates/owns the subject; Decorator adds behavior to an already-existing object) or with Adapter (Adapter changes the interface; Decorator keeps the same interface). Another mistake is decorators that break the Liskov substitution guarantee by changing the contract rather than augmenting it.

Summary — what a strong answer emphasizes

Decorator = composition-based, runtime, stackable extension of behavior through a shared interface. It beats inheritance precisely where inheritance fails: many orthogonal, optional features whose combinations you don't want to enumerate, and behavior you want to vary per-object at runtime. Cite the class-explosion motivation, the OCP benefit, a canonical example (Java I/O or coffee pricing), and the Proxy/Adapter distinctions — that combination signals real depth.

**Comparison:**

```
Aspect	Inheritance (subclassing)	Decorator (composition)
Binding time	Static, compile-time	Dynamic, runtime
Relationship	is-a (subclass of parent)	is-a interface + has-a wrapped object
Combining N optional features	~2^N classes (explosion)	N decorators, composed freely
Coupling	Tight to parent (fragile base class)	Loose — depends only on shared interface
Scope of change	Whole class / all instances	A single object instance
Adding a new feature	New subclass, may touch hierarchy	New decorator, existing code untouched (OCP)
Order of features	Fixed by hierarchy	Caller-chosen (order can matter)
Similar pattern	How it differs from Decorator
Adapter	Changes the interface to fit a client; Decorator keeps the same interface
Proxy	Controls access / lifecycle of the subject; Decorator adds behavior to an existing subject
Strategy	Swaps an algorithm via a field; Decorator wraps and layers whole objects
```

**Code:**

```

// Component: common interface interface Coffee { double cost(); String description(); }

// ConcreteComponent: the base object class Espresso implements Coffee { public double cost() { return 2.00; } public String description() { return "Espresso"; } }

// Decorator: abstract wrapper holding a Coffee and forwarding to it abstract class CoffeeDecorator implements Coffee { protected final Coffee inner; // has-a Coffee (composition) protected CoffeeDecorator(Coffee inner) { this.inner = inner; } }

// ConcreteDecorators: add behavior before/after delegating class Milk extends CoffeeDecorator { public Milk(Coffee c) { super(c); } public double cost() { return inner.cost() + 0.50; } public String description() { return inner.description() + " + Milk"; } }

class Sugar extends CoffeeDecorator { public Sugar(Coffee c) { super(c); } public double cost() { return inner.cost() + 0.20; } public String description() { return inner.description() + " + Sugar"; } }

// Client: compose freely at RUNTIME — no new subclass needed public class Demo { public static void main(String[] args) { Coffee order = new Sugar(new Milk(new Espresso())); System.out.println(order.description() + " = $" + order.cost()); } }

CODE
Copy

```java
// Contrast: the inheritance approach that Decorator avoids.
// Every combination is its own class -> class explosion (2^n).
class EspressoWithMilk extends Espresso { /* override cost/desc */ }
class EspressoWithMilkAndSugar extends Espresso { /* ... */ }
class EspressoWithSugar extends Espresso { /* ... */ }
// ...add Whip, Caramel -> dozens of classes, all fixed at compile time.
```

**Example input / output:**

```text
Input  (build): new Sugar(new Milk(new Espresso()))
Trace:
  Sugar.cost()  -> inner.cost() + 0.20
    Milk.cost() -> inner.cost() + 0.50
      Espresso.cost() -> 2.00
    Milk  = 2.00 + 0.50 = 2.50
  Sugar = 2.50 + 0.20 = 2.70

Output (description): "Espresso + Milk + Sugar"
Output (cost):        2.70

Add a new topping (e.g., Whip) = ONE new decorator class,
no change to Espresso/Milk/Sugar or the client (Open/Closed).
```

**Flow:**

1. wraps (has-a)
2. «interface»
3. Coffee
4. +cost() : double
5. +description() : String
6. Espresso
7. +cost() : double
8. +description() : String
9. «abstract»
10. CoffeeDecorator
11. #inner : Coffee
12. Milk
13. +cost() : double
14. +description() : String
15. Sugar
16. +cost() : double
17. +description() : String

