---
type: "practice"
difficulty: "Deep"
---

# How would you design classes to be extensible for a new feature without editing existing ones?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Over-engineering vs Simplicity](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

The literal ask is: how do I design classes so that adding a new feature means writing new code, not editing existing, tested code? The interviewer is probing whether I actually understand the Open/Closed Principle (OCP) — "open for extension, closed for modification" — and, more importantly, whether I know how to achieve it in practice (via abstraction + polymorphism + composition) and, crucially, when not to (because premature abstraction is its own bug factory). At "deep understanding" level for an SDE/SDE-2/3/Architect, a strong answer must go past reciting the acronym: it must name the concrete mechanisms (interfaces/strategy/plugin registries/DI), show a before/after refactor, discuss the trade-offs, and state the honest limits of the principle.

Clarifying assumptions I'd state: OCP applies at the seam where variation is expected — not everywhere. So my design goal is to identify the axis of change the feature request implies (a new payment method? a new export format? a new discount rule?) and put a stable abstraction across that axis. I'll assume we control the code, use a typed OO language (Java/TypeScript-style examples), and have tests.

Plan: (1) define what "editing existing classes" costs and why we avoid it, (2) the core techniques, (3) a concrete refactor from a switch to polymorphism, (4) the enabling patterns (Strategy, Factory, plugin registry, DI), (5) trade-offs and when not to do it.

Why avoid editing existing classes

Every time you edit a working, deployed class you risk regressing behavior that other callers depend on, you must re-test it fully, and in a large team you create merge contention on a hot file. If instead a new feature is a new class that plugs into a stable extension point, the blast radius is tiny: existing code keeps its tests green, review is scoped, and two teams can add features in parallel without touching the same lines. That is the practical payoff of OCP — not architectural purity, but reduced risk and parallelism.

The core mechanism: program to an abstraction

OCP is achieved by depending on an interface (or abstract base) instead of a concrete class, and by using polymorphism so the caller doesn't branch on type. When a new variant appears, you add a new implementation of the interface; the calling code is untouched.

The classic smell that OCP fixes is a growing if/else or switch on a type field:

JAVA
// BAD — every new shape/type forces an edit here
double area(Shape s) {
    if (s.type == CIRCLE)    return Math.PI * s.r * s.r;
    else if (s.type == RECT) return s.w * s.h;
    // add TRIANGLE -> must edit this method, its tests, redeploy
}

Every new shape modifies this method. The OCP version pushes the varying behavior into the type:

JAVA
interface Shape { double area(); }
class Circle implements Shape { public double area() { return Math.PI*r*r; } }
class Rect   implements Shape { public double area() { return w*h; } }
// New feature = new class Triangle implements Shape {} — area() is never touched.

The general recipe: find the thing that will vary, name it as an interface, and let each variant be a class. New feature = new class.

Enabling patterns

Several GoF patterns are just OCP made concrete for a particular kind of variation:

Strategy — swap an algorithm behind an interface (e.g., PricingStrategy, TaxCalculator). New pricing rule = new strategy class.
Factory / Abstract Factory — centralize object creation so the creation site isn't a growing switch either; the factory maps a key to a constructor.
Plugin / Registry — implementations self-register into a map/registry at startup (via DI, annotations, or a service loader). The dispatcher looks the handler up by key; adding a handler needs zero edits to the dispatcher.
Decorator — add behavior (logging, caching, retry) around an existing object without editing it.
Template Method / Observer — extend fixed algorithms by overriding hook steps, or by attaching new listeners.

The most powerful for "no edits ever" is the registry + dependency injection combo: the coordinator depends on Map<Key, Handler> (or a List<Handler> it filters), and the DI container injects all discovered implementations. A new handler is a new file with an annotation — genuinely zero modification of existing code.

A realistic refactor (payments / notifications)

Say we start with a notification service that switches on channel:

JAVA
switch (channel) {
  case EMAIL: sendEmail(...); break;
  case SMS:   sendSms(...);   break;
}

Refactor to a Notifier interface, one class per channel, and a registry keyed by channel. Adding WhatsApp becomes: write WhatsAppNotifier implements Notifier, register it. The NotificationService, its tests, and every other channel stay byte-for-byte identical. This is exactly the pattern I'd use for payment gateways, file exporters (CSV/PDF/XLSX), discount rules, or auth providers — anywhere the business keeps adding "one more of the same kind of thing."

#### Trade-offs
and the honest limits (deep-understanding differentiator)

OCP is not free and is easy to over-apply:

You cannot make code open to every future change. You choose an axis. If tomorrow's requirement varies along a different axis than the one you abstracted, you'll still edit code — and now through an awkward abstraction. This is why senior engineers say "wait for the second or third variant before abstracting." Speculative interfaces created for a single implementation are the #1 form of over-engineering.

Abstraction has a cost: more classes, indirection, harder to read the actual control flow, more moving parts for a newcomer. If a feature genuinely requires behavioral change to existing logic (not a new variant), forcing it into an extension point is worse than a clean edit.

Extension points can leak. If new implementations frequently need the interface itself to grow new methods, you didn't find the right seam — every extension then does force edits to every implementation (an ISP problem).

Data/schema and cross-cutting changes (a new column, a new required field) often can't be made purely additive; OCP is about behavioral variation, not every conceivable change.

My rule of thumb: apply OCP where the domain tells you variation is coming (payment methods, notification channels, report formats, pluggable rules) and you already have ≥2 variants. Elsewhere, keep it simple and refactor to an interface at the moment the second variant arrives (the "Rule of Three"). That refactor is cheap precisely because tests already exist.

Summary — what a strong answer emphasizes
Restate it as OCP: closed for modification, open for extension, achieved via abstraction + polymorphism + composition, not editing.
Show the switch → interface/one-class-per-variant refactor; that's the money demonstration.
Name the enabling patterns (Strategy, Factory, registry + DI/plugin) and pick registry+DI as the truly zero-edit approach.
Give a real domain (payments/notifications/exporters) so it's concrete.
Crucially, show judgment: OCP is per-axis, costs indirection, and should follow the Rule of Three — over-abstracting is a defect. That "when NOT to" is what separates a deep answer from a memorized one.

Common follow-ups: "Isn't the factory now the thing you edit?" (→ move creation to a self-registering registry). "How does DI help?" (→ inject all implementations, dispatcher stays fixed). "When would you knowingly violate OCP?" (→ single variant, or a genuine behavioral change; don't build a plugin for one plugin).

**Comparison:**

```
Technique	Variation it absorbs	New feature = ?	Cost / risk
Interface + polymorphism	"Another of the same kind" (shapes, channels)	New implementing class	Extra classes; must pick the right seam
Strategy	Swappable algorithm (pricing, tax)	New strategy class	Overkill for one algorithm
Factory / Abstract Factory	Object creation of variants	New branch in factory (or make it a registry)	Factory itself can become a switch
Registry + DI / plugin	Any keyed dispatch	New self-registering class (zero edits)	DI/registry machinery, discovery magic
Decorator	Add-on behavior (log/cache/retry)	New wrapper class	Wrapping order matters; deep stacks confuse
Template Method	Fixed algorithm, varying steps	Override hook method in subclass	Inheritance coupling
Do abstract for OCP when…	Do NOT (keep it simple) when…
You already have ≥2 variants (Rule of Three)	Only one variant exists (speculative interface = over-engineering)
The domain clearly keeps adding "one more" (gateways, formats, rules)	The change is a genuine behavioral fix to existing logic
Multiple teams add variants in parallel	The abstraction would hide, not clarify, control flow
The interface is stable (implementations rarely force it to grow)	Each new variant keeps forcing new interface methods (wrong seam / ISP smell)
```

**Code:**

```

// ---- Extension point: one stable interface ---- public interface Notifier { Channel channel(); void send(Message m); }

// ---- Each feature is a NEW class; existing ones never edited ---- public class EmailNotifier implements Notifier { public Channel channel() { return Channel.EMAIL; } public void send(Message m) { /* SMTP / } } public class SmsNotifier implements Notifier { public Channel channel() { return Channel.SMS; } public void send(Message m) { / SMS gateway / } } // NEW FEATURE = just add this file. Nothing else changes. public class WhatsAppNotifier implements Notifier { public Channel channel() { return Channel.WHATSAPP; } public void send(Message m) { / WhatsApp API */ } }

// ---- Registry + DI: coordinator is CLOSED for modification ---- public class NotificationService { private final Map<Channel, Notifier> byChannel;

CODE
Copy
// DI container injects ALL discovered Notifier beans; no switch here.
public NotificationService(List<Notifier> notifiers) {
    this.byChannel = notifiers.stream()
        .collect(Collectors.toMap(Notifier::channel, n -> n));
}

public void notify(Channel c, Message m) {
    Notifier n = byChannel.get(c);
    if (n == null) throw new IllegalArgumentException("No notifier for " + c);
    n.send(m);   // polymorphism — never a type switch
}

}

CODE
Copy

```typescript
// Same idea in TypeScript with a self-registering registry (plugin style)
interface Exporter { format(): string; export(data: Row[]): Buffer; }

const registry = new Map<string, Exporter>();
export function register(e: Exporter) { registry.set(e.format(), e); }

class CsvExporter implements Exporter {
  format() { return "csv"; }
  export(data: Row[]) { /* ... */ return Buffer.from(""); }
}
register(new CsvExporter());
// New feature: create PdfExporter, call register(new PdfExporter()). Dispatcher untouched.

export function exportAs(fmt: string, data: Row[]): Buffer {
  const e = registry.get(fmt);
  if (!e) throw new Error(`Unsupported format ${fmt}`);
  return e.export(data);
}
```

**Example input / output:**

```text
Scenario: product ships EMAIL + SMS. Then Marketing asks for WhatsApp.

Without OCP (switch-based):
- Input: "add WHATSAPP channel"
- Change set: edit `NotificationService.notify()` switch, edit its unit tests, re-run + re-review the whole class, redeploy the coordinator. Merge conflict risk if another team is also editing that switch.

With OCP (registry + DI, code above):
- Input: "add WHATSAPP channel"
- Change set: add ONE new file `WhatsAppNotifier.java`. `NotificationService` diff = 0 lines. Its existing tests stay green untouched.
- Runtime trace: `notify(WHATSAPP, m)` → `byChannel.get(WHATSAPP)` → `WhatsAppNotifier.send(m)`. No branch on type, no edit to shared code.

Output/observable result: same feature delivered, but blast radius drops from "1 hot shared class + its tests" to "1 brand-new isolated class," enabling parallel work and green regressions.
```

**Flow:**

1. notify(channel, msg)
2. lookup by key
3. polymorphic send
4. Caller
5. NotificationServiceCLOSED for modification
6. (Registry / DI mapChannel -> Notifier)
7. EmailNotifier
8. SmsNotifier
9. WhatsAppNotifierNEW - only added file
10. Notifier.send

