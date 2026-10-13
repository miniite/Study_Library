---
type: "practice"
difficulty: "Expert"
---

# Design the reconciliation subsystem for a payments platform that matches internal ledger entries against daily PSP settlement files — how do you detect missing captured-but-not-settled and settled-but-not-recorded transactions and drive them to resolution?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Payment Ledgers](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Payment_Ledgers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

The goal is to design a high-level reconciliation subsystem that ensures all transactions in an internal ledger are matched against daily PSP (Payment Service Provider) settlement files, detecting any discrepancies such as missing captured-but-not-settled and settled-but-not-recorded transactions. The key ideas involve implementing robust data ingestion, comparison logic, and a feedback loop for resolution.

#### Triage

The question probes our ability to design a scalable and reliable reconciliation system that can handle large volumes of transaction data efficiently. We need to ensure the subsystem can ingest PSP settlement files, compare them with internal ledger entries, detect mismatches, and provide mechanisms for resolving those discrepancies. Key assumptions include the availability of PSP settlement files on a daily basis and the need for high accuracy in matching transactions.

#### First-Principles Solution

Data Ingestion:

PSP Settlement Files: Design a process to ingest PSP settlement files into our system. This could be done using batch jobs or real-time ingestion depending on the scale.
Internal Ledger Entries: Ensure all internal ledger entries are stored in a database with a unique identifier for each transaction.

Transaction Matching:

Hashing and Indexing: Implement hashing (e.g., SHA-256) of transactions to create unique identifiers that can be used for rapid comparison.
Indexing: Create indexes on the ledger entries based on common fields like transaction ID, timestamp, amount, etc. This speeds up the matching process.

Discrepancy Detection:

Matching Logic: Use a combination of exact matches (e.g., transaction IDs) and fuzzy matching (e.g., comparing timestamps and amounts). This ensures that both settled-but-not-recorded and captured-but-not-settled transactions are identified.
Conflict Resolution Rules: Define rules for resolving conflicts. For example, if the PSP has a transaction record but there is no match in the ledger, it may indicate a missing recorded transaction.

Feedback Loop:

Alerts and Notifications: Set up notifications for discrepancies so that relevant stakeholders are alerted.
Resolution Mechanisms: Provide mechanisms to update the internal ledger with any new or corrected transactions identified through the reconciliation process.

Monitoring and Auditing:

Performance Metrics: Monitor the performance of the system, such as throughput, response time, and error rates.
Audit Logs: Maintain detailed logs of all reconciliations for auditing purposes.

#### Closing Summary

The high-level design for the reconciliation subsystem involves a combination of data ingestion from PSP settlement files, robust transaction matching logic, conflict resolution mechanisms, and feedback loops to drive discrepancies towards resolution. The key is ensuring that the system can handle large volumes of data efficiently while maintaining accuracy and reliability.

#### Common Mistakes/Follow-Ups
Overlooking Performance: A common mistake is underestimating the performance requirements. Ensure the system can scale with increasing transaction volumes.
Lack of Indexing: Failing to use proper indexing for quick comparison could lead to slow reconciliation processes.
Inadequate Conflict Resolution: Not having clear and effective conflict resolution rules may result in manual interventions, slowing down the process.

**Comparison:**

| Transaction ID | Matched? |
|----------------|----------|
| 101            | Yes      |
| 102            | No       |
| 103            | No       |

**Code:**

```python
# Example of a simple transaction matching function
def match_transactions(psp_settlement_file, ledger_entries):
    psp_hashes = {hash_transaction(entry): entry for entry in psp_settlement_file}
    ledger_hashes = {hash_transaction(entry): entry for entry in ledger_entries}

    # Find matches and mismatches
    matched_transactions = set(psp_hashes.keys()) & set(ledger_hashes.keys())
    missing_psp_transactions = set(psp_hashes.keys()) - matched_transactions
    missing_ledger_transactions = set(ledger_hashes.keys()) - matched_transactions

    return matched_transactions, missing_psp_transactions, missing_ledger_transactions

def hash_transaction(entry):
    # Simple hashing function for demonstration
    return hashlib.sha256(json.dumps(entry).encode()).hexdigest()
```

**Example input / output:**

```text
Input:
PSPSettlementFile = [
    {"transaction_id": 101, "timestamp": "2023-10-01T10:00:00Z", "amount": 50},
    {"transaction_id": 102, "timestamp": "2023-10-01T11:00:00Z", "amount": 75}
]
LedgerEntries = [
    {"transaction_id": 101, "timestamp": "2023-10-01T10:00:00Z", "amount": 50},
    {"transaction_id": 103, "timestamp": "2023-10-01T12:00:00Z", "amount": 100}
]

Output:
Matched Transactions: [101]
Missing PSP Transactions: [102]
Missing Ledger Entries: [103]
```

**Data:**

```json
{
  "PSPSettlementFile": [
    {"transaction_id": 101, "timestamp": "2023-10-01T10:00:00Z", "amount": 50},
    {"transaction_id": 102, "timestamp": "2023-10-01T11:00:00Z", "amount": 75}
  ],
  "LedgerEntries": [
    {"transaction_id": 101, "timestamp": "2023-10-01T10:00:00Z", "amount": 50},
    {"transaction_id": 103, "timestamp": "2023-10-01T12:00:00Z", "amount": 100}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Ingest PSP Settlement File
4. Is Matching Possible?
5. Generate Hashes and Indexes
6. Handle Error/Retry
7. Match Transactions
8. Identify Discrepancies
9. Alerts for Discrepancies
10. Conflict Resolution Rules
11. Update Ledger
12. Monitor and Audit

