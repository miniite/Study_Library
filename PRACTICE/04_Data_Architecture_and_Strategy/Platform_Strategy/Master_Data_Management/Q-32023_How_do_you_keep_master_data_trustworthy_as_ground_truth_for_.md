---
type: "practice"
difficulty: "Expert"
---

# How do you keep master data trustworthy as ground truth for a RAG/agent system — what happens to answers when the golden record is wrong, and how do you gate LLM access to master data?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Records and Survivorship](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/Golden_Records_and_Survivorship.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that master data remains trustworthy as the ground truth for a Retrieval-Augmented Generation (RAG) or agent system, we need to establish robust mechanisms for validating and updating our "golden record" of master data. When the golden record is wrong, this can lead to misleading or inaccurate answers from LLMs, so gating LLM access to master data is critical. This involves several steps:

Define the Golden Record: Identify the authoritative source(s) that contain the most accurate and complete information.
Validation Mechanisms: Implement rules and processes to validate new or updated data against existing records.
Automated Updates: Use automated systems to regularly update master data based on validated sources.
Access Control: Restrict direct access to the master data while ensuring LLMs can still query it through controlled APIs.

Let's break this down step-by-step:

#### Triaging the Question

The question is asking how we maintain the integrity of our master data, particularly when using LLMs in a RAG system. The core issue here is ensuring that the data used by the LLM is always correct and up-to-date. If the golden record (the most accurate version of the truth) is wrong, it can negatively impact the performance and reliability of the LLM.

#### First-Principles Solution
Define the Golden Record: Start by identifying the sources that contain the most accurate data. For instance, in a financial context, transactions from a core banking system might be considered the golden record.
Validation Mechanisms:
Implement rules and processes to validate new or updated data against existing records. This could involve complex validation logic based on business rules or machine learning models that detect anomalies.
Automated Updates: Use automated systems (like ETL processes) to periodically update the master data from trusted sources. For example, a job might run nightly to sync fresh transaction data from banking systems into the master database.
Access Control:
Gate LLM access to the master data by providing it through secure APIs that validate requests and ensure only correct data is accessed.
Use data masking techniques if necessary to protect sensitive information while still allowing relevant fields to be queried.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, maintaining trustworthy master data involves a combination of defining clear golden records, robust validation mechanisms, automated updates, and secure access controls. Without these measures, the LLMs may provide inaccurate answers, leading to poor user experiences or critical errors in decision-making processes.

#### Common Mistakes

Failing to define clear golden records.
Not implementing adequate validation mechanisms.
Relying solely on manual updates rather than automated processes.
Overlooking security and access controls, which can lead to data breaches or unauthorized use of sensitive information.

#### Example Scenario

Imagine a financial institution where the master data includes customer transaction histories. If there is a discrepancy in one of these records (e.g., an incorrect amount), this could affect how LLMs respond to queries about a customer’s financial status. To mitigate this, we would ensure that all transactions are validated against known good sources before being updated in the master database.

Closing

This approach ensures that the data used by LLMs is always accurate and up-to-date, thereby enhancing the reliability of responses generated for users or agents. Regular audits and feedback loops can further improve the quality of our master data over time.

**Comparison:**

```
Transaction ID	Amount (USD)	Timestamp
12345	-100	2023-10-01T14:00:00Z
67890	500	2023-10-02T15:00:00Z

Note: Transaction ID 12345 is marked for invalidation due to the negative amount.
```

**Code:**

```python
# Example validation function in Python
def validate_transaction(transaction_data):
    # Simple validation logic (could be complex in practice)
    if transaction_data['amount'] < 0:
        return False
    return True

# Example update process
def update_master_data(new_transactions):
    for txn in new_transactions:
        if validate_transaction(txn):
            master_db.update(txn)  # Assuming 'master_db' is a database object
```

**Example input / output:**

```text
Input:
Transaction data for ID 12345 with an invalid negative amount.

Output:
Validation failed. Transaction not updated in master database.
```

**Data:**

```json
{
  "transaction": {
    "id": 12345,
    "amount": -100, // Invalid transaction amount
    "timestamp": "2023-10-01T14:00:00Z"
  }
}
```

**Flow:**

1. Valid
2. Invalid
3. Start
4. Validate New Data
5. Update Master DB
6. Log and Reject
7. Notify Stewards
8. End

