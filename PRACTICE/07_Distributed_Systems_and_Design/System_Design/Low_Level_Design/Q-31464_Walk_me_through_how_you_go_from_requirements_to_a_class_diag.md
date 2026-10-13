---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through how you go from requirements to a class diagram step by step.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composition vs Inheritance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Composition_vs_Inheritance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

This is the natural sibling of "how do you find classes" — here the interviewer wants the end-to-end procedure that takes a requirements document and lands on a UML class diagram: not just the classes, but their attributes, methods, and — crucially — the relationships and multiplicities between them. For an intermediate candidate they're checking that I (1) have a repeatable, ordered method; (2) understand UML relationship types (association, aggregation, composition, inheritance, dependency) and when each applies; and (3) can validate the diagram against the use cases and iterate. They also want to see design maturity: SRP, cohesion/coupling, and knowing when to stop.

Clarifying questions I'd ask: What are the primary use cases and their scale? Which parts must be extensible? Is this a self-contained library or a service with persistence? These decide how much abstraction (interfaces, strategies) I bake in. Plan: I'll present a numbered walk — requirements → actors/use cases → noun/verb extraction → filter classes → attributes & methods → relationships & multiplicities → apply principles/patterns → validate & iterate — and demonstrate it on a small booking example (design a movie-ticket booking / seat reservation), producing an actual class diagram.

Step 1 — Capture requirements and use cases

I start by writing crisp functional use cases as actor + action + outcome: "A user searches shows, selects seats, and books; the system holds the seats, takes payment, and issues a booking." I list non-functional needs (concurrency on seat holds, extensibility of payment methods). This grounds everything: the diagram must make every use case executable, and NFRs justify abstractions. Skipping this is the #1 cause of diagrams that look tidy but can't run the actual flow.

Step 2 — Identify actors and boundaries

I separate actors (User, Admin, PaymentGateway) from the system under design. Actors that are merely callers usually aren't classes I implement; external systems (PaymentGateway) become interfaces/adapters at the boundary. This keeps the core domain clean and testable.

Step 3 — Noun/verb extraction (candidate members)

The mechanical pass: nouns → candidate classes/attributes, verbs → candidate methods/relationships. From the booking sentence: nouns = user, show, seat, booking, payment, movie, screen, theatre; verbs = search, select, book, hold, pay, issue. This is a deliberately broad draft to be pruned.

Step 4 — Filter into real domain classes

I apply the same filters as class discovery: keep a noun as a class only if it has identity + owned state (Show, Seat, Booking); demote value-like nouns to fields (a seat's row/number are attributes); drop external actors; merge synonyms. The result is a lean class list. I prefer fewer, cohesive classes to many anemic ones.

Step 5 — Assign attributes and methods (responsibilities)

For each surviving class I give it the data it owns and the behavior over that data — high cohesion. This is where SRP is enforced: one reason to change per class.

Seat — row, number, status (AVAILABLE/HELD/BOOKED); can be held/booked/released.
Show — movie, screen, startTime, seat map; can list available seats.
Booking — user, show, chosen seats, amount, status; created at checkout.
BookingService — orchestrates: hold seats → take payment → confirm. It delegates payment.
PaymentGateway (interface) — pay(amount) → result; concrete RazorpayGateway, StripeGateway.

I deliberately pull orchestration into BookingService and payment behind an interface, so seat logic, booking logic, and payment logic each change for independent reasons.

Step 6 — Establish relationships and multiplicities (the UML heart)

Now I connect classes with the correct UML relationship — this is what elevates a class list into a class diagram:

Composition (filled diamond) — a strong "owns/part-of" where the part can't exist without the whole and shares its lifecycle: Show ◆— Seat map is arguably composition; Theatre ◆— Screen.
Aggregation (hollow diamond) — "has-a" but independent lifecycle: Booking o— Seats (seats outlive a booking).
Association — a plain "uses/references": Booking — User, Booking — Show, with multiplicities (a User 1—* Booking; a Booking 1—1..* Seat).
Inheritance (hollow triangle) — "is-a", only when subtypes truly differ in behavior. I'd resist making PremiumSeat/RegularSeat subclasses if a type enum + price suffices.
Dependency (dashed arrow) — transient "uses in a method": BookingService ..> PaymentGateway.

I always annotate multiplicities (1, 0..1, 1..*, *) because they encode business rules ("a booking has at least one seat", "a show belongs to exactly one screen"). Getting these right is a strong-candidate signal.

Step 7 — Apply principles and patterns where earned

With the skeleton drawn, I introduce patterns only where responsibilities call for them: Strategy for pricing (weekday/weekend), Adapter/interface for payment providers, State for seat status transitions if they get complex, Factory if creation is non-trivial. Patterns follow from responsibilities; I don't start with them. I keep favoring composition over inheritance and dependence on abstractions (interfaces) to stay Open/Closed.

Step 8 — Validate against use cases and iterate

Finally I re-walk every use case across the diagram like a sequence trace: "search → Show.availableSeats(); select+book → BookingService.book(user, show, seats) → Seat.hold() → PaymentGateway.pay() → Booking confirmed → Seat.book()." If a step has no home, a class or method is missing; if one class does the whole flow, it's a God class — split it. I iterate until every use case flows cleanly and each class has one reason to change. Then the diagram is done.

Summary — what a strong answer emphasizes

The walk is: requirements/use cases → actors & boundaries → noun/verb extraction → filter to domain classes → attributes+methods (SRP) → relationships with correct UML type + multiplicities → patterns where earned → validate by tracing every use case and iterate. The differentiators at intermediate level are (a) choosing the right relationship (composition vs aggregation vs association vs inheritance) and annotating multiplicities, and (b) treating the diagram as something I validate against the use cases and refine, not a one-shot artifact.

#### Common mistakes
& follow-up probes

Mistakes: modeling every noun as a class; omitting multiplicities; overusing inheritance; a God Manager class holding all logic; drawing associations without a direction/meaning. Likely follow-ups: "Composition vs aggregation here?" (lifecycle test — does the part die with the whole?); "Where does the concurrent seat-hold live?" (isolated in Seat.hold()/service with locking or optimistic status check); "Add a new payment provider?" (implement the PaymentGateway interface — no change to BookingService, proving Open/Closed).

**Comparison:**

```
UML relationship	Symbol	Meaning	Example here	Multiplicity
Composition	filled diamond ◆	Part can't outlive whole; shared lifecycle	Theatre ◆— Screen, Show ◆— Seat map	1 — 1..*
Aggregation	hollow diamond ◇	Has-a, independent lifecycle	Booking ◇— Seat	1 — 1..*
Association	solid line	Uses/references	Booking — User, Booking — Show	* — 1
Inheritance	hollow triangle △	Is-a, distinct behavior	avoided (enum instead)	—
Dependency	dashed arrow ⇢	Transient use in a method	BookingService ⇢ PaymentGateway	—
Step	Input	Output	Key check
1 Requirements	Prose	Use cases + NFRs	Every flow captured
3 Noun/verb	Use cases	Candidate members	Broad draft
4 Filter	Candidates	Domain classes	Identity + owned state
5 Members	Classes	Attributes + methods	SRP / cohesion
6 Relationships	Classes	UML edges + multiplicities	Right relationship type
8 Validate	Diagram	Verified diagram	Trace every use case
```

**Code:**

```java
// The class diagram below, expressed as Java, after walking the steps.

enum SeatStatus { AVAILABLE, HELD, BOOKED }

class Seat {                          // owns seat state + transitions (SRP)
    final String row; final int number;
    private SeatStatus status = SeatStatus.AVAILABLE;
    Seat(String row, int number) { this.row = row; this.number = number; }
    synchronized boolean hold() {     // concurrency concern lives here
        if (status != SeatStatus.AVAILABLE) return false;
        status = SeatStatus.HELD; return true;
    }
    void book()    { status = SeatStatus.BOOKED; }
    void release() { status = SeatStatus.AVAILABLE; }
    boolean isAvailable() { return status == SeatStatus.AVAILABLE; }
}

class Show {                          // composition of seats; knows availability
    final String movie; final String screen; final long startTime;
    final List<Seat> seats;
    Show(String movie, String screen, long startTime, List<Seat> seats) {
        this.movie = movie; this.screen = screen; this.startTime = startTime; this.seats = seats;
    }
    List<Seat> availableSeats() { return seats.stream().filter(Seat::isAvailable).toList(); }
}

interface PaymentGateway { boolean pay(long amount); }   // boundary abstraction

class Booking {                       // association to User/Show, aggregation of Seats
    final String user; final Show show; final List<Seat> seats; final long amount;
    enum Status { PENDING, CONFIRMED, FAILED } Status status = Status.PENDING;
    Booking(String user, Show show, List<Seat> seats, long amount) {
        this.user = user; this.show = show; this.seats = seats; this.amount = amount;
    }
}

class BookingService {                // orchestration only; delegates payment
    private final PaymentGateway gateway;
    BookingService(PaymentGateway gateway) { this.gateway = gateway; }

    Booking book(String user, Show show, List<Seat> chosen, long amount) {
        for (Seat s : chosen) if (!s.hold()) { chosen.forEach(Seat::release); throw new IllegalStateException("Seat taken"); }
        Booking b = new Booking(user, show, chosen, amount);
        if (gateway.pay(amount)) { chosen.forEach(Seat::book); b.status = Booking.Status.CONFIRMED; }
        else { chosen.forEach(Seat::release); b.status = Booking.Status.FAILED; }
        return b;
    }
}
```

**Example input / output:**

```

Use-case trace validating the diagram (this is Step 8):

CODE
Copy
Requirement sentence:
  "A user searches shows, selects seats, books; system holds seats, takes payment, issues booking."

Trace across the diagram/code:
  search    -> Show.availableSeats()                 [Show provides it -> OK]
  select    -> user picks List<Seat>                 [-> BookingService.book(...)]
  hold      -> Seat.hold() per seat (synchronized)   [concurrency handled in Seat -> OK]
  pay       -> PaymentGateway.pay(amount)            [delegated, provider-agnostic -> OK]
  issue     -> new Booking(status=CONFIRMED)         [Booking owns booking state -> OK]

Sample run:
  Show s = new Show("Dune", "Screen-1", t, [A1,A2,A3(HELD)]);
  bookingService.book("u1", s, [A1, A2], 500);
    A1.hold() -> true ; A2.hold() -> true ; gateway.pay(500) -> true
    A1.book(), A2.book(); booking.status = CONFIRMED
  => Booking{user=u1, seats=[A1,A2], amount=500, status=CONFIRMED}

  Contention case:
  bookingService.book("u2", s, [A2]);   // A2 now BOOKED
    A2.hold() -> false  => release all, throw "Seat taken"
```

**Flow:**

1. missing home / God class
2. all flows clean, SRP holds
3. Requirements doc
4. Write use cases + NFRs
5. Identify actors & systemboundary
6. Noun/verb extraction
7. Filter -> domain classes
8. Attributes + methods per classSRP
9. Relationships:composition/aggregation/association/inheritance/dependency + multiplicities
10. Apply patterns where earned
11. Trace every use case acrossdiagram
12. Class diagram complete
13. composition
14. aggregation
15. association
16. dependency
17. creates
18. 1
19. 1
20. *
21. 1..*
22. 1..*
23. 1
24. Show
25. +availableSeats()
26. Seat
27. -status
28. +hold()
29. +book()
30. +release()
31. Booking
32. -status
33. BookingService
34. +book()
35. «interface»
36. PaymentGateway
37. +pay(amount)

