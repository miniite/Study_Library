---
type: "practice"
difficulty: "Deep"
---

# A VP challenges you to justify why the org should invest in a shared money/currency type when "floats have worked fine"; walk through the concrete production failures you'd cite and how you'd pilot the change to prove it out before a broad rollout.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Open-closed Extensibility](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Open-closed_Extensibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the VP's challenge, I'll provide an overview of why investing in a shared money/currency type might be beneficial over using floats. We’ll then delve into specific concrete production failures and outline a pilot plan to validate the change.

TL;DR:

Investing in a shared money/currency type is crucial to mitigate issues like precision loss, performance bottlenecks, and data integrity problems. By piloting the new system, we can demonstrate its benefits and build confidence before a broader rollout.

#### Triaging the Question

The VP is questioning why we should adopt a shared money/currency type when floats have been working "fine" so far. The key points being probed are:

Precision Issues: Floats can lead to precision errors, particularly in financial calculations.
Performance and Scalability: Complex financial operations might be slower with float-based systems.
Data Integrity and Compliance: Precision issues can affect data integrity, leading to non-compliance.

#### Assumptions
The current system uses floats for money/currency representations.
The organization deals with high-frequency transactions or large volumes of monetary calculations.
There is a need for high precision in financial operations.
Step-by-Step Solution from First Principles
1. Precision Issues

Floats are binary fractions, which means they can represent some fractional values exactly but not others (e.g., 0.1). This leads to rounding errors that accumulate over multiple operations, causing discrepancies. For instance, in financial systems dealing with millions of transactions daily, these small errors can compound and become significant.

2. Performance Impact

High-frequency financial transactions require precise calculations. Using floats for financial calculations can lead to performance bottlenecks as the system may need to handle additional logic to correct rounding errors, which slows down transaction processing.

3. Data Integrity and Compliance

Precision issues can result in data inconsistencies that might violate regulatory requirements or internal policies. Ensuring accurate financial records is critical, especially under stringent compliance environments like those governed by financial regulations (e.g., SEC, FCA).

Pilot Plan

To justify the investment, we'll design a pilot plan to demonstrate the benefits of using a shared money/currency type.

1. Define Requirements
Identify critical business processes that are prone to precision issues.
Determine the accuracy requirements for these processes.
2. Select Technology
Choose an appropriate data type (e.g., Decimal in Java) or library (e.g., BigDecimal in Java) for handling money/currency.
3. Design and Implement
Develop a prototype with the chosen technology.
Integrate this prototype into a subset of critical business processes to test its performance and accuracy.
4. Monitor and Measure
Track key metrics such as transaction success rates, error rates, and processing times before and after implementation.
Compare the results to identify improvements in precision and performance.

#### Closing Summary

By adopting a shared money/currency type, we can address precision issues, improve performance, and ensure data integrity. A well-planned pilot will provide evidence of these benefits and help build confidence for a broader rollout.

#### Common Mistakes
and Follow-Ups
Failing to fully understand the precision issues in current systems.
Overlooking potential performance improvements that could be gained.
Not properly defining requirements and measuring outcomes during the pilot phase.

Piloting the change will allow us to address these concerns effectively and demonstrate clear benefits, thus justifying the investment.

**Code:**

```java
public class Money {
    private BigDecimal amount;

    public Money(double amount) {
        this.amount = new BigDecimal(String.valueOf(amount));
    }

    public BigDecimal getAmount() {
        return amount;
    }
}
```

**Flow:**

1. Define Requirements
2. Select Technology
3. Design and ImplementPrototype
4. Monitor Performance andAccuracy
5. Analyze Results
6. Pilot Go/No-Go Decision
7. Broad Rollout Plan

