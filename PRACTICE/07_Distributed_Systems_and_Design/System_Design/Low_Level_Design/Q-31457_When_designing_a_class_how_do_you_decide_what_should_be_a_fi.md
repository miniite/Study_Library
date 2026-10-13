---
type: "practice"
difficulty: "Basic"
---

# When designing a class how do you decide what should be a field vs a parameter vs a method?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composition vs Inheritance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Composition_vs_Inheritance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked? When you sit down to design a class, you have to place every piece of data and behaviour somewhere. This question probes whether you understand the three homes a value can live in — a field (instance state), a parameter (input passed into a method or constructor), or a method (behaviour) — and whether you have a principled way to decide, rather than dumping everything into fields. It is a foundational LLD/OOP question. The interviewer wants to see that you understand object lifetime, state, encapsulation, and the difference between "what an object is" versus "what an object does" versus "what it needs to be told each time".

Clarifying assumptions: I'll assume a typical OOP language (Java/C#/TypeScript/Python) and an object that models a real concept (e.g., ShoppingCart, BankAccount). The same reasoning applies broadly.

Plan: Define each of the three homes, give the decision rule for each, walk a concrete example (a BankAccount) to show the reasoning live, then summarise with a quick decision checklist and common mistakes.

First principles: three homes for a value

1. A field (instance variable) = long-lived state the object owns. A field belongs to the object for its whole lifetime. Ask: "Is this value part of the object's identity or state that must persist between method calls?" If yes → field. A BankAccount must remember its balance between a deposit() today and a withdraw() tomorrow, so balance is a field. Fields answer "what the object IS / remembers."

2. A parameter = short-lived input supplied for a single operation. A parameter lives only for the duration of one call. Ask: "Does this value vary per call and only matters during that one operation?" If yes → parameter. The amount you deposit changes every time and is meaningless after the call returns, so deposit(amount) takes it as a parameter — you would NOT store amount as a field. Parameters answer "what the object is TOLD, right now."

3. A method = behaviour the object performs. A method is a verb — an operation that reads/mutates state or computes a result. Ask: "Is this an action or a derived value the object can perform/compute?" If yes → method. deposit, withdraw, getBalance are methods.

The decision rules (how I actually decide)
Field if: the value must survive across multiple method calls, is part of the object's identity/configuration, and is genuinely owned by this object. It represents state.
Parameter if: the value is provided by the caller, differs per invocation, or is a collaborator the method needs only transiently. Prefer a parameter over a field when the value is not persistent state — this keeps the object smaller and the method self-contained and testable.

#### Cons
tructor parameter → field: a special, important case. Values needed for the object's whole life (mandatory dependencies, immutable identity like accountNumber, injected collaborators like a Logger) come in as constructor parameters and get stored into fields. This is dependency injection.
Method (behaviour) if: it's an action, a state transition, or a computed/derived value. Prefer a method over a field for anything derivable. If fullName can be computed from firstName + lastName, expose it as a method/getter, not a stored field — storing it risks it going stale (two sources of truth).
Worked reasoning: designing BankAccount
accountNumber → identity, set once, needed forever → constructor parameter stored as a final field.
balance → mutable state that must persist between deposits/withdrawals → field.
deposit(amount) / withdraw(amount) → the amount changes each call and is transient → method parameters, not fields.
getBalance() → an action returning state → method.
interestForDays(days) → days is caller input per call → parameter; the result is derived → method, not a stored field (avoid staleness).
A TransactionLogger the account uses → a collaborator needed for the whole life → constructor parameter → field (inject it). If it were needed by only one method occasionally, I'd pass it as a method parameter instead.

#### Trade-offs
/ judgment
Fewer fields is usually better. Every field is state you must keep consistent, thread-safe, and serialize. If something can be a parameter or a computed method, prefer that — it shrinks the object's surface area and mutable state.
Field vs parameter is a lifetime question; field vs method is a "stored vs derived" question. Keep those two decisions separate in your head.
Encapsulation: fields should generally be private; expose behaviour through methods, not raw fields, so the class controls its invariants (e.g., balance can never go negative because only withdraw touches it and it checks).
Configuration threshold: if a method starts taking 5+ parameters that are always the same for a given object, that's a signal those belong in fields (set via constructor) — the method has too many transient inputs that are really object configuration.

#### Common mistakes
/ follow-up probes
Turning transient inputs (amount, requestId) into fields → creates accidental state and bugs across calls.
Storing derived values as fields → staleness / two sources of truth.
Making everything a public field with no methods → an anemic data bag with no encapsulation.

#### Follow-up
an interviewer may ask: "When would you pass a dependency as a method parameter vs inject it as a field?" Answer: inject as a field when it's used broadly and for the object's whole life; pass as a method parameter when only one method needs it or it varies per call (keeps coupling low).
Summary — what a strong answer emphasises

A field = persistent state / identity the object owns (lives for the object's lifetime). A parameter = transient per-call input (lives for one call). A method = behaviour or a derived value. Decide by lifetime (survives calls → field; per-call → parameter) and by derivable-vs-stored (computable → method, don't store). Prefer fewer fields, inject long-lived dependencies through the constructor, and expose derived values as methods to avoid stale state.

**Comparison:**

```
Home	Lifetime	Decide it when...	Example in BankAccount
Field	Whole object lifetime	Value is persistent state / identity / long-lived dependency	balance, accountNumber, logger
Parameter	One method/ctor call	Value is caller input, varies per call, transient	amount, days, annualRatePct
Method	N/A (it's behaviour)	It's an action, state transition, or a derivable value	deposit(), getBalance(), interestForDays()
```

**Code:**

```java
public final class BankAccount {
    // FIELDS: identity + long-lived state the object owns for its whole life
    private final String accountNumber;        // identity, set once (constructor param -> final field)
    private final TransactionLogger logger;    // injected collaborator, used for whole life
    private long balanceInPaise;               // mutable state, persists across calls

    // Constructor PARAMETERS become fields (dependency injection + identity)
    public BankAccount(String accountNumber, TransactionLogger logger) {
        this.accountNumber = accountNumber;
        this.logger = logger;
        this.balanceInPaise = 0;
    }

    // METHOD with a PARAMETER: `amount` is transient, varies per call -> NOT a field
    public void deposit(long amountInPaise) {
        if (amountInPaise <= 0) throw new IllegalArgumentException("amount must be positive");
        balanceInPaise += amountInPaise;
        logger.record(accountNumber, "DEPOSIT", amountInPaise);
    }

    public void withdraw(long amountInPaise) {
        if (amountInPaise > balanceInPaise) throw new IllegalStateException("insufficient funds");
        balanceInPaise -= amountInPaise;      // invariant: never negative, enforced here
    }

    // METHOD returning STATE
    public long getBalanceInPaise() { return balanceInPaise; }

    // DERIVED value -> METHOD with a PARAMETER, never a stored field (avoids staleness)
    public long interestForDays(int days, double annualRatePct) {
        return Math.round(balanceInPaise * (annualRatePct / 100.0) * days / 365.0);
    }
}
```

**Example input / output:**

```text
Reasoning trace for each value in `BankAccount`:

| value        | question asked                                  | home              |
|--------------|-------------------------------------------------|-------------------|
| accountNumber| identity, needed for whole life?  -> yes         | field (from ctor) |
| logger       | collaborator, used broadly for whole life? -> yes| field (injected)  |
| balance      | must survive across deposit/withdraw? -> yes     | field             |
| amount       | changes each call, transient? -> yes             | parameter         |
| days / rate  | caller input per call? -> yes                    | parameter         |
| interest     | derivable from balance? -> yes                    | method (not field)|

Sample run:
- new BankAccount("ACC-1", logger)  -> balance = 0
- deposit(100000)                   -> balance = 100000 (amount not stored)
- withdraw(30000)                   -> balance = 70000
- interestForDays(365, 6.0)         -> returns 4200 (computed, nothing stored)
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. A value to place in the class
6. Is it behaviour / an action orderivable result?
7. Make it a METHOD
8. Must it survive across multiplecalls or define identity?
9. Make it a FIELD; if supplied atcreation, take as constructorparameter
10. Make it a method PARAMETER- transient per-call input

