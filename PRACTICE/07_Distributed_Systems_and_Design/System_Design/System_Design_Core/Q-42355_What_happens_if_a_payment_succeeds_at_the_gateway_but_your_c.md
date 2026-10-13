---
type: "practice"
difficulty: "Intermediate"
---

# What happens if a payment succeeds at the gateway but your callback never arrives?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Payment Reconciliation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a payment is successful at the gateway but no callback confirms it, several issues can arise. The core problem involves ensuring that we detect and handle this scenario to maintain system integrity and prevent financial discrepancies. Let's break down the situation from first principles.

#### Triage

The question essentially asks about what happens when a payment transaction succeeds on an external gateway, but our application doesn't receive any confirmation or callback due to various potential issues like network problems, service outages, or misconfigurations in either the gateway or our system. The key aspects being probed are:

Detection and Handling: How we can detect if such a failure has occurred.
Impact on System State: What state does the payment assume (e.g., pending, failed) when no confirmation is received?
Risk Management: Ensuring that users aren't charged twice and that our system maintains consistency.

#### Assumptions
The gateway provides an asynchronous callback mechanism to notify us of a transaction's success or failure.
Our backend has some form of transaction state management, such as using databases or distributed systems like Redis.
There are known methods for handling retries and fallbacks in case the initial callback doesn't arrive.

#### First-Principles Solution
Transaction State Management: Maintain a consistent state of each payment request either by storing it in a database (e.g., MySQL) or using a distributed cache (e.g., Redis).
Callback Confirmation Mechanism:
When a transaction is initiated, create a corresponding entry in the system's state.
The gateway sends a callback to confirm success; this is stored as confirmation data for that specific transaction ID.
Retry Logic: Implement retry logic on both client and server sides to ensure callbacks are received even if they are delayed due to network issues or temporary outages.
Timed Expiry: Add a timeout mechanism so transactions can be marked as failed if no callback is received within a certain timeframe (e.g., 30 minutes).
Fallback Mechanisms: Design fallbacks for critical operations, such as automatically marking the transaction as failed after retries and expiry.
Step-by-Step Solution
Create Transaction Entry:
When initiating a payment request, create an entry in the database (or distributed cache) with status PENDING.
Send Payment Request to Gateway:
Initiate the payment request and ensure it has a unique transaction ID.
Gateway Confirmation Handling:
The gateway sends back a confirmation callback.
Update the transaction's state to COMPLETED in the system if the callback is received successfully.
Implement Retry Logic:
If no response is received within a reasonable time (e.g., 5 minutes), mark the transaction as pending and initiate a retry process.
Handle retries with exponential backoff to avoid overwhelming the gateway.
Timed Expiry:
After multiple retries fail, set up a timeout mechanism that marks transactions as FAILED if no confirmation is received within a fixed time window (e.g., 30 minutes).
Fallback Mechanisms:
Implement fallback logic to ensure users are not charged twice.
Send notifications or alerts for failed transactions.

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake: Assuming the gateway will always send a callback, leading to incomplete state management in the system. Always implement retry mechanisms and timeouts.

#### Follow-Up
Question: How would you handle scenarios where both the gateway and your system experience outages simultaneously?
Further Consideration: Integrating with third-party services like Stripe or PayPal requires understanding their specific APIs and behaviors.

**Comparison:**

```
Transaction ID	Status
tx_123456	PENDING
tx_789012	COMPLETED
tx_234567	FAILED
```

**Code:**

```python
# Example of a transaction state update in Python using SQLAlchemy

from sqlalchemy import create_engine, Column, Integer, String, Enum
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class PaymentTransaction(Base):
    __tablename__ = 'payment_transactions'

    id = Column(Integer, primary_key=True)
    transaction_id = Column(String, unique=True)
    status = Column(Enum('PENDING', 'COMPLETED', 'FAILED'), default='PENDING')

engine = create_engine('sqlite:///payments.db')
Session = sessionmaker(bind=engine)

def initiate_payment(transaction_id):
    session = Session()
    new_transaction = PaymentTransaction(transaction_id=transaction_id, status='PENDING')
    session.add(new_transaction)
    session.commit()

# Example of updating transaction status
def update_transaction_status(transaction_id, status):
    session = Session()
    transaction = session.query(PaymentTransaction).filter_by(transaction_id=transaction_id).first()
    if transaction:
        transaction.status = status
        session.commit()
```

**Example input / output:**

```text
Input:
- Initiate payment request with transaction ID: tx_123456

Output:
- Transaction entry in database: {id: 1, transaction_id: 'tx_123456', status: 'PENDING'}

Callback (Simulated):
- Gateway sends successful callback for tx_123456

Output:
- Update transaction status to COMPLETED: {id: 1, transaction_id: 'tx_123456', status: 'COMPLETED'}

Timeout Expiry:
- No callback received within 30 minutes

Output:
- Mark transaction as FAILED due to timeout: {id: 1, transaction_id: 'tx_123456', status: 'FAILED'}
```

**Data:**

```json
{
  "payment_transactions": [
    {
      "id": 1,
      "transaction_id": "tx_123456",
      "status": "PENDING"
    },
    {
      "id": 2,
      "transaction_id": "tx_789012",
      "status": "COMPLETED"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Initiate Payment Request
6. Gateway Callback?
7. Mark Transaction asCompleted
8. Retry After 5 Minutes?
9. Update Status to Failed Due toTimeout
10. Transaction in Pending State

