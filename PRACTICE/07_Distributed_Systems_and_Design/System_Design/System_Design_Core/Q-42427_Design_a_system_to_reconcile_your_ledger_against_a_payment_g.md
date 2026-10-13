---
type: "practice"
difficulty: "Deep"
---

# Design a system to reconcile your ledger against a payment gateway's settlement report daily. How do you handle missing callbacks, duplicates, and amounts that don't match?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Payment Reconciliation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust system for reconciling your ledger against a payment gateway’s settlement report, we need to ensure accuracy, reliability, and efficiency. The primary goal is to identify discrepancies, handle missing or duplicate callbacks, and manage amounts that don’t match. This task involves several key components: data ingestion, validation, reconciliation logic, and reporting.

#### Triage

The question is asking for a detailed design of the system's architecture, including how it will process incoming settlement reports from a payment gateway and reconcile them with internal ledger records. The main concerns are:

Handling missing callbacks (i.e., transactions that were not recorded in our system).
Managing duplicate entries.
Resolving discrepancies in amounts.

#### Assumptions
Transactions are uniquely identifiable by an ID.
Both the ledger and payment gateway have a timestamp for each transaction.
The settlement report contains all necessary details for reconciliation (e.g., transaction amount, timestamp).
Step-by-Step Solution
1. Data Ingestion

The system needs to ingest the daily settlement reports from the payment gateway. This can be done through an API call or file upload mechanism.

Payment Gateway SettlementReport
Ingestion Service
Database
2. Validation and Duplication Handling

Before processing, validate that each transaction in the settlement report is unique by checking against existing records.

PYTHON
def validate_transaction(transaction):
    # Check if transaction ID exists in database
    return not db.exists(f"transaction:{transaction['id']}")

If a duplicate is found, update the ledger with any new information (e.g., correct amount) and log the duplicate entry.

3. Reconciliation Logic

Reconcile each transaction from the settlement report against the corresponding record in the ledger.

Payment Gateway SettlementReport
Validate & Handle Duplicates
Reconcile Transactions

For transactions that match, update their status to "reconciled." For those that don't match (e.g., amounts), log the discrepancy and notify relevant stakeholders.

4. Missing Callbacks

Identify missing callbacks by comparing timestamps in the settlement report with existing ledger entries.

Payment Gateway SettlementReport
Check Timestamps AgainstLedger

If a transaction is missing, it should be marked as such and investigated further to understand why the callback was not received.

5. Reporting

Generate reports on the reconciliation status for each day, highlighting any discrepancies or issues that need attention.

Reconcile Transactions
Generate Reports

#### Closing Summary

By implementing a robust system design with clear steps for data ingestion, validation, reconciliation, and reporting, we can ensure accurate and reliable ledger reconciliation against the payment gateway. This design addresses missing callbacks, duplicates, and amount discrepancies through systematic processes.

#### Common Mistakes/Follow-Ups
Not Handling Duplicates Properly: Failing to update existing records or log duplicate entries.
Ignoring Missing Callbacks: Not actively seeking out transactions that were not received.
Inadequate Reporting Mechanism: Lack of detailed reporting on reconciliation status and discrepancies.

**Comparison:**

| Transaction ID | Status       | Reason                  |
|----------------|--------------|-------------------------|
| 1024           | Discrepancy  | Amount Mismatch         |
| 1025           | Duplicate    | Already in Ledger       |

**Code:**

```python
def validate_transaction(transaction):
    # Check if transaction ID exists in database
    return not db.exists(f"transaction:{transaction['id']}")

def reconcile_transactions(settlement_report, ledger_db):
    for report_transaction in settlement_report:
        if validate_transaction(report_transaction):
            # Handle duplicate logic here
            pass

        # Reconciliation logic
        ledger_transaction = get_ledger_transaction_by_id(report_transaction['id'])

        if ledger_transaction:
            if ledger_transaction['amount'] != report_transaction['amount']:
                log_discrepancy(ledger_transaction, report_transaction)

    # Mark all transactions as reconciled
```

**Example input / output:**

| Transaction ID | Amount (Ledger) | Amount (Settlement Report) |
|----------------|----------------|----------------------------|
| 1024           | $6.00          | $5.99                      |
| 1025           | $8.00          | $7.99                      |

**Data:**

```yaml
settlement_report:
  - id: "1024"
    amount: 5.99
    timestamp: "2023-10-01T09:00:00Z"
  - id: "1025"
    amount: 7.99
    timestamp: "2023-10-01T09:15:00Z"
```

**Flow:**

1. Payment Gateway SettlementReport
2. Ingestion Service
3. Database
4. Validate & Handle Duplicates
5. Reconcile Transactions
6. Generate Reports

