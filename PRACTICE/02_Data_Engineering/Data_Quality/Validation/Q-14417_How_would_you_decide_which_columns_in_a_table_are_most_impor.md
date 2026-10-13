---
type: "practice"
difficulty: "Intermediate"
---

# How would you decide which columns in a table are most important to validate first?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validating Loads and Deduplication](<../../../../02_Data_Engineering/Data_Quality/Validation/Validating_Loads_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To determine which columns in a table should be validated first, consider factors like business impact, data dependency, and historical quality issues. We'll use a step-by-step approach starting from basic principles to ensure we cover all important aspects.

#### Triage
Understanding the Question

The interviewer is probing your understanding of how to prioritize validation efforts for database tables based on their significance to the overall data ecosystem. This involves assessing the potential impact of errors in different columns and the dependencies between them.

Step-by-Step Solution

Identify Key Business Metrics: Start by listing all the business metrics that rely heavily on specific columns. For example, if a column is used in financial calculations or customer analytics, it will have a higher business impact.

Assess Data Dependency: Columns with more dependencies on other parts of the system (e.g., through joins, aggregations) should be prioritized as they can cascade issues throughout the data flow.

Historical Quality Analysis: Analyze historical data to identify columns that frequently contain errors or inconsistencies. This might involve looking at log files, audit trails, and error reports from previous validation runs.

Risk Assessment: Evaluate the potential impact of a failure in each column on downstream processes and systems. For instance, if a critical report is generated using this data, any issues could have severe consequences.

User Feedback: Engage with stakeholders to understand their pain points and feedback regarding data quality issues. This can provide insights into which columns are causing the most problems.

Automation Considerations: Prioritize columns that are easier to automate validation for, as this reduces manual effort and improves consistency over time.

#### Example
Workflow

Let's say we have a customer table with several columns: customer_id, first_name, last_name, email, address. We can prioritize these based on the following steps:

Business Impact:

first_name and last_name: These are crucial for personal identification.
email: Often used for communication, which could affect customer support.

Data Dependency:

customer_id: Used in many other tables; errors here would ripple through the entire database.

Historical Quality Analysis:

Review logs and error reports to see if any of these columns are flagged more frequently for issues like null values or typos.

Risk Assessment:

customer_id is riskiest because it can cause cascading errors in related tables.
email could affect customer service but might be easier to validate.

User Feedback:

Stakeholders might complain about missing names more often than other issues, highlighting their priority.

Automation Considerations:

Automate basic validation for customer_id and first_name as they are critical yet straightforward.

#### Closing Summary

By systematically analyzing each column based on business impact, data dependency, historical quality issues, risk assessment, user feedback, and automation considerations, we can prioritize which columns to validate first. This ensures that our efforts are focused where they will have the most significant positive impact.

#### Common Mistakes/Follow-Ups
Ignoring Business Impact: Failing to consider the business significance of each column.
Overcomplicating Automated Validation: Assuming complex validation logic is easier or more effective than simpler checks.
Not Considering Historical Data: Missing out on insights from past data quality issues.

If asked, you could discuss how this approach might change in a larger organization with more complex dependencies and stakeholder requirements.

**Comparison:**

| Column | Business Impact | Data Dependency | Historical Quality Issues |
|--------|-----------------|-----------------|--------------------------|
| customer_id | High            | Critical        | Medium                    |
| first_name  | High            | Low             | High                      |
| last_name   | Medium          | Low             | Medium                    |
| email      | Low             | Low             | Medium                    |
| address    | Low             | Low             | Low                       |

**Code:**

```python
# Example code to identify high-risk columns based on business rules

def prioritize_columns(table_data):
    priorities = {
        'customer_id': 1,
        'first_name': 2,
        'last_name': 3,
        'email': 4,
        'address': 5
    }

    # Simulate a basic risk assessment function
    def assess_risk(column, data):
        # Placeholder logic for assessing risk
        return len([row for row in data if column in row and not is_valid(row[column])])

    prioritized_columns = sorted(priorities.items(), key=lambda x: (assess_risk(x[0], table_data), priorities[x[0]]))

    return [col[0] for col in prioritized_columns]
```

**Example input / output:**

```text
# Example input and output

table_data = {
  "customer_id": ["12345", null, "67890"],
  "first_name": ["John", "Peter", "Sarah"],
  "last_name": ["Doe", null, "Smith"],
  "email": ["john@example.com", "peter@invalid", "sarah@domain.com"],
  "address": ["123 Main St", "456 Elm St", "789 Oak St"]
}

prioritized_columns = prioritize_columns(table_data)
print(prioritized_columns)  # Output: ['customer_id', 'first_name', 'last_name', 'email', 'address']
```

**Data:**

```json
{
  "customer_id": ["12345", null, "67890"],
  "first_name": ["John", "Peter", "Sarah"],
  "last_name": ["Doe", null, "Smith"],
  "email": ["john@example.com", "peter@invalid", "sarah@domain.com"],
  "address": ["123 Main St", "456 Elm St", "789 Oak St"]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is column used in criticalbusiness processes?
5. Assess historical quality issues
6. Consider data dependency anduser feedback
7. Automate validation wherepossible
8. Prioritize columns with highestrisk
9. End
10. Rank based on dependenciesand business impact
11. Consider automation forsimpler checks

