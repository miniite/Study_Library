---
type: "practice"
difficulty: "Expert"
---

# Design graceful degradation for a fintech app where view-balance may serve stale data but transfer-money must fail closed. Explain how you encode dependency criticality into the architecture so degradation is deliberate, and how you continuously test the degraded paths in production.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Static Stability](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Static_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design graceful degradation for a fintech app where view-balance may serve stale data but transfer-money must fail closed, we need to ensure that critical operations (like transfers) are isolated from non-critical ones (like balance views). This involves understanding the system architecture, encoding dependency criticality, and implementing robust testing strategies.

The key idea is to prioritize operations based on their criticality. In this case, while stale data in the view-balance operation can cause minor inconvenience, a failed transfer-money operation could lead to significant financial losses or legal issues. Therefore, it's crucial that transfers fail gracefully rather than risk a failure.

#### Triage
What We're Probing

The interviewer is probing your understanding of system architecture principles, specifically around graceful degradation and the trade-offs between availability and correctness in distributed systems. They want to see how you balance these priorities while ensuring fault tolerance and resilience.

#### Assumptions
The app consists of multiple microservices.
Stale data can be cached or served from a read replica.
Transfer-money operations require real-time data, but view-balance does not.
There is a need for continuous testing to validate the degradation strategy in production.

#### Solution
First-Principles Approach
Step 1: Designing Service Dependencies

To encode dependency criticality, we can use a service mesh or API gateway to manage calls between microservices. This allows us to prioritize and control the flow of requests based on their criticality. For instance:

User Request
API Gateway
Balance View Service
Transfer Money Service
Database Replica
Real-time Database

The API gateway acts as a central point of control, ensuring that critical operations (like transfer-money) are prioritized and non-critical ones (like balance view) can serve stale data.

Step 2: Implementing Stale Data Handling

For the Balance View Service:

Use read replicas or caching mechanisms to handle stale data.
Ensure these services have fallback paths in case of primary service failure.
User Request
API Gateway
Read Replica
Step 3: Failing Closed for Critical Operations

For the Transfer Money Service:

Ensure this service uses real-time data from the database.
Implement a fail-fast mechanism where any failure results in an immediate rollback or rejection of the transaction.
User Request
API Gateway
Real-time Database
Transaction Manager
Step 4: Continuous Testing and Monitoring

To ensure that degradation is deliberate and effective, we need to implement continuous testing in production:

Use canary releases to test the degraded paths.
Set up automated monitoring and alerting for critical operations.
Regularly simulate failures to validate the resilience of the system.

#### Closing Summary

By encoding dependency criticality through a service mesh or API gateway and implementing fail-fast mechanisms, we ensure that non-critical operations like balance views can serve stale data while critical operations like transfers fail closed. This approach requires careful design and continuous testing but provides a robust solution for maintaining high availability in a distributed system.

#### Common Mistakes/Follow-Ups
Not Prioritizing Critical Operations: Failing to prioritize critical operations properly can lead to significant risks.
Lack of Fallback Mechanisms: Not having proper fallback mechanisms for non-critical services can result in user dissatisfaction or data inconsistencies.
Insufficient Testing: Continuous testing and monitoring are crucial but often overlooked, leading to undetected failures.

**Comparison:**

| Operation | Service       | Data Freshness | Degradation Strategy |
|-----------|--------------|----------------|---------------------|
| Balance View | Read Replica | Stale          | Serve stale data    |
| Transfer Money | Real-time DB | Real-Time      | Fail closed         |

**Code:**

```python
# Example API Gateway Configuration (using Istio)
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: balance-view-service
spec:
  host: balance-view-service.default.svc.cluster.local
  subsets:
  - name: v1
    labels:
      version: v1

---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: balance-view-virtualservice
spec:
  hosts:
  - "*"
  gateways:
  - gateway-egress
  http:
  - match:
    - uri:
        prefix: /balance-view/
    route:
    - destination:
        host: balance-view-service.default.svc.cluster.local
        subset: v1
```

**Example input / output:**

```text
Request:
GET /balance-view/balance?account=123456

Response (Stale Data):
{
    "balance": 999.99,
    "lastUpdated": "2023-10-01T12:00:00Z"
}

Request:
POST /transfer-money
{
    "amount": 100.00,
    "fromAccount": 789456,
    "toAccount": 123456
}

Response (Fail Closed):
{
    "status": "FAIL",
    "message": "Insufficient Funds or Internal Error"
}
```

**Data:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: transfer-money-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: transfer-money-service
  template:
    metadata:
      labels:
        app: transfer-money-service
    spec:
      containers:
      - name: transfer-money-container
        image: transfer-money-app:v1
        ports:
        - containerPort: 8080
```

**Flow:**

1. User Request
2. API Gateway
3. Balance View Service (ReadReplica)
4. Transfer Money Service(Real-time DB)
5. Database Read-Only
6. Database Real-Time

