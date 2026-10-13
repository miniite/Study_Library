---
type: "practice"
difficulty: "Intermediate"
---

# Describe a case where you used a security policy or predicate to restrict which rows a batch job could touch and how you tested the negative cases.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database Access Control and Auditing](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a Database Developer with significant experience, I often find myself in scenarios where ensuring data integrity and confidentiality is paramount, especially when dealing with large-scale batch jobs that process sensitive information. One such case involved implementing security policies to restrict access based on specific criteria.

#### Triaging the Question

The core of this question revolves around how you implemented a security policy or predicate to control which rows a batch job can interact with, and how you validated its effectiveness by testing negative cases (i.e., scenarios where the policy should prevent certain actions).

#### First-Principles Solution

In this project, we were tasked with processing financial transactions from multiple branches of a large multinational corporation. Each branch had its own set of rules regarding which transactions could be processed based on various factors like transaction type, amount, and time.

Step 1: Define the Security Policy

We defined our security policy using SQL Predicates in PostgreSQL, specifically leveraging the CHECK constraint within triggers or directly as part of the query logic. For example:

SQL
CREATE OR REPLACE FUNCTION validate_transaction()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.transaction_type = 'international' AND NEW.amount > 1000 THEN
        RETURN NULL; -- Reject the transaction
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER restrict_international_transactions
BEFORE INSERT ON transactions
FOR EACH ROW EXECUTE FUNCTION validate_transaction();
Step 2: Implementing Negative Testing Cases

To ensure our security policy was robust, we designed a series of negative test cases. These tests aimed to cover various scenarios where the predicate should fail but did not.

Transaction Amount: We created a set of transactions with amounts just below and above the threshold.
Transaction Type: We simulated both valid and invalid transaction types (e.g., international, domestic).
Time Constraints: We also included time-based restrictions, ensuring that certain actions were only allowed during specific hours.
Step 3: Execution and Validation

We ran these tests using a combination of automated scripts and manual checks. The key was to ensure that our batch jobs strictly adhered to the defined security policies without any exceptions.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, by carefully defining security policies and rigorously testing them through negative cases, we ensured that sensitive data remained protected. Common mistakes include not thoroughly considering edge cases or overlooking potential bypass mechanisms. Future improvements might involve leveraging advanced techniques like row-level security (RLS) in PostgreSQL for enhanced flexibility.

#### Follow-up
questions could delve deeper into the specifics of the SQL implementation or explore how this approach scales to handle more complex scenarios.

**Code:**

```sql
CREATE OR REPLACE FUNCTION validate_transaction()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.transaction_type = 'international' AND NEW.amount > 1000 THEN
        RETURN NULL; -- Reject the transaction
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER restrict_international_transactions
BEFORE INSERT ON transactions
FOR EACH ROW EXECUTE FUNCTION validate_transaction();
```

