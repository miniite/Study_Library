---
type: "practice"
difficulty: "Deep"
---

# Design the architecture for a BNPL (buy-now-pay-later) product spanning eligibility real-time credit decisioning disbursement and repayment schedules — how do you keep the credit-decision path fast and the money-movement ledger correct across all three legs?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Payment Ledgers](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Payment_Ledgers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an architecture for a BNPL product that balances real-time credit decisioning with accurate money movement, we need to ensure that the credit-decision path remains fast and efficient while maintaining the integrity of the financial ledger across eligibility assessment, disbursement, and repayment. This requires a multi-faceted approach involving microservices, event-driven architectures, and robust data management.

#### Triaging the Question

The core challenge here is ensuring that the real-time credit decisioning process does not compromise on speed but still maintains accuracy and consistency in financial transactions. We need to design systems that can handle high throughput and low latency while ensuring data integrity.

#### Assumptions
High Throughput: The system must support a large number of concurrent requests.
Low Latency: Real-time credit decisions are crucial, so the decisioning process needs to be fast.
Data Consistency: Financial transactions need to be accurately recorded and reflect real-time changes.
Step-by-Step Solution
1. Eligibility Assessment

The first step is to design a microservice for eligibility assessment. This service will check the customer's creditworthiness in near-real time.

Microservice Design:
Input: Customer data (e.g., credit score, income, past payment history).
Output: Eligibility decision and credit limit.
Technology Stack: Use a lightweight microservice framework like Spring Boot or FastAPI. This service should be stateless to ensure scalability.
2. Credit Decisioning

Once the eligibility assessment is completed, we need to make real-time decisions on whether to approve the transaction.

Real-Time Decisioning:
Service: Use a dedicated microservice that can handle high request rates and provide quick responses.
Technology Stack: Consider using cloud-native services like AWS Lambda or Google Cloud Functions. These offer serverless capabilities which are ideal for handling bursty traffic patterns.
Database: Utilize in-memory databases (e.g., Redis) to store temporary data and decision rules.
3. Disbursement

The next step is to disburse the funds to the customer's account once the credit decision is approved.

Disbursement Microservice:
Input: Transaction details, approval response from real-time decisioning.
Output: Updated financial ledger reflecting the disbursed amount.
Technology Stack: Use a reliable transactional database (e.g., PostgreSQL) to ensure atomicity and consistency. Leverage distributed ledgers like Ethereum or Hyperledger for more complex transactions.
4. Repayment Scheduling

Finally, we need to set up repayment schedules and handle the periodic payments from customers.

Repayment Scheduling:
Service: A background service that updates customer accounts and sends reminders.
Technology Stack: Use a task queue like Amazon SQS or Apache Kafka to manage asynchronous tasks. This ensures that reminders are sent out reliably without blocking the main application flow.
Data Management: Store repayment plans and transaction histories in a scalable database.

#### Closing Summary

To keep the credit-decision path fast, we leverage serverless and microservice architectures with high throughput and low latency capabilities. For accurate money movement, we use robust data management practices to ensure consistency across all financial transactions. By breaking down the process into discrete services, we can achieve both speed and accuracy while maintaining system reliability.

#### Common Mistakes/Follow-Ups

Mistake: Overcomplicating the decisioning logic by keeping it stateful.

#### Follow-Up
Ensure that decisioning services are stateless to allow for easy scaling and reduced complexity.

Mistake: Ignoring data consistency in favor of speed.

#### Follow-Up
Implement robust transactional controls and use distributed ledgers where appropriate.

**Comparison:**

| Service | Input                 | Output               |
|---------|-----------------------|----------------------|
| Eligibility Assessment | Customer Data         | Eligibility Decision |
| Credit Decisioning      | Approval Request      | Disbursement Details |
| Disbursement            | Transaction Details   | Ledger Update        |
| Repayment Scheduling    | Scheduled Payments    | Payment Log          |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Eligibility Assessment] --> B[Credit Decisioning];
    B --> C[Disbursement];
    C --> D[Repayment Scheduling];
```

**Example input / output:**

```text
Input:
- Eligibility Assessment Request (JSON)

Output:
- Eligibility Decision Response (JSON)

Example Request/Response:

Request:
{
  "customer_id": 12345,
  "credit_score": 780,
  "income": 90000,
  "past_payments": [
    {"status": "paid", "amount": 150},
    {"status": "late", "amount": 200}
  ]
}

Response:
{
  "approved": true,
  "credit_limit": 3000
}
```

**Data:**

```json
{
  "eligibility_assessment": {
    "customer_id": 12345,
    "credit_score": 780,
    "income": 90000,
    "past_payments": [
      {"status": "paid", "amount": 150},
      {"status": "late", "amount": 200}
    ]
  },
  "decisioning_response": {
    "approved": true,
    "credit_limit": 3000
  }
}
```

**Flow:**

1. Customer
2. Eligibility Service
3. Credit Decisioning Service
4. Disbursement Service
5. Repayment Scheduling Service
6. Ledger Update

