---
type: "practice"
difficulty: "Deep"
---

# How do you decide between attribute-level trust scores propagated to consumers vs a single binary 'validated' flag on the golden record — what do downstream consumers actually need to act safely?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Records and Survivorship](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Golden_Records_and_Survivorship.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between attribute-level trust scores and a single binary "validated" flag, we must consider the needs of downstream consumers who require safe actions based on data integrity. Attribute-level trust scores provide granular insights into the reliability of each piece of data, enabling more nuanced decision-making. A single "validated" flag simplifies the decision process but may not capture complex data quality issues.

To determine which approach is better suited, we should first understand that downstream consumers need a balance between simplicity and accuracy. We will start by considering the trade-offs involved in both methods.

Attribute-Level Trust Scores:

These scores provide detailed information about the reliability of individual attributes.
They allow for more precise validation rules, such as cross-attribute consistency checks or threshold-based validations.
However, they can be complex to implement and require a robust trust scoring mechanism, which may involve additional computational overhead.

Single Binary "Validated" Flag:

This flag offers a binary decision: whether the golden record is valid or not.
It simplifies consumer logic by reducing decisions to yes/no questions.
However, it might fail to capture nuanced data quality issues and could result in incorrect decisions if the golden record contains invalid attributes.

To make an informed decision, we must consider several factors:

#### Assumptions

#### Cons
umer Complexity: Some consumers may require detailed insights into each attribute's reliability, while others need a simpler validation flag.
Data Quality Issues: How prevalent are data quality issues in your environment? High-frequency issues might necessitate more granular trust scores.
Performance Constraints: Do downstream systems have performance constraints that could be affected by complex logic?
Step-by-Step Solution

Identify Downstream Consumer Needs:

Analyze the requirements of each consumer to understand their level of complexity and decision-making capabilities.
For example, a financial system might require detailed trust scores for risk assessment, whereas an HR system may only need a simple "validated" flag.

Evaluate Data Quality Issues:

Assess the prevalence and severity of data quality issues in your environment.

#### Cons
ider implementing a trust scoring model that can handle varying levels of data reliability.

Implement Both Approaches:

Develop a prototype with both attribute-level trust scores and a binary "validated" flag.
Compare their performance, accuracy, and ease of use across different consumers.

Conduct A/B Testing:

Deploy the prototypes in a controlled environment to gather real-world data.
Analyze how each approach affects downstream consumer behavior and decision-making processes.

Optimize for Performance:

Ensure that attribute-level trust scores are computed efficiently, considering caching strategies or summarization techniques.
If performance is an issue with the binary flag approach, consider a hybrid solution where critical attributes use detailed validation while others rely on simpler flags.

Monitor and Iterate:

Continuously monitor the effectiveness of both approaches in production.
Gather feedback from consumers to refine and improve the MDM strategy over time.

#### Closing Summary

Ultimately, the decision between attribute-level trust scores and a single binary "validated" flag depends on the specific needs of downstream consumers. Attribute-level scores offer more granular insights but come with added complexity. A binary validated flag simplifies decisions but may fail to address nuanced data quality issues. By evaluating consumer requirements, conducting thorough testing, and continuously optimizing, we can ensure that our MDM strategy supports safe and effective data consumption.

#### Common Mistakes/Follow-Ups
Overlooking Consumer Needs: Failing to consider the complexity of downstream systems when designing trust scoring mechanisms.
Ignoring Data Quality Issues: Not adequately addressing prevalent data quality issues in the initial design phase.
Neglecting Performance Constraints: Designing solutions that do not account for performance constraints, leading to suboptimal system behavior.

**Comparison:**

| Attribute | Trust Score |
|-----------|-------------|
| name      | 0.95        |
| age       | 0.80        |
| address   | 0.70        |

**Code:**

```python
# Example pseudocode for trust score computation and validation flag generation

def compute_trust_scores(record):
    # Logic to compute attribute-level trust scores
    return {attr_name: score for attr_name, score in record.items()}

def generate_validation_flag(trust_scores):
    # Logic to determine overall validity based on trust scores
    if all(score >= threshold for score in trust_scores.values()):
        return True
    else:
        return False
```

**Example input / output:**

```text
Input:
{
  "record": {
    "name": "John Doe",
    "age": 30,
    "address": "123 Main St, Anytown, USA"
  },
  "trust_scores": {
    "name": 0.95,
    "age": 0.80,
    "address": 0.70
  }
}

Output:
{
  "validation_flag": True,
  "trust_scores": {
    "name": 0.95,
    "age": 0.80,
    "address": 0.70
  }
}
```

**Data:**

```json
{
  "record": {
    "name": "John Doe",
    "age": 30,
    "address": "123 Main St, Anytown, USA"
  },
  "trust_scores": {
    "name": 0.95,
    "age": 0.80,
    "address": 0.70
  }
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Start
5. Is consumer complexity high?
6. Maintain attribute-level trustscores
7. Monitor and optimizeperformance
8. D
9. Consider hybrid approach forcritical attributes
10. Do data quality issues exist?
11. Implement detailed trustscoring model
12. Simple validation logic suffices

