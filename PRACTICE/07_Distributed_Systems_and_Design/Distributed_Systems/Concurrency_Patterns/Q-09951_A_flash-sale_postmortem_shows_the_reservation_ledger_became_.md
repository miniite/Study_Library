---
type: "practice"
difficulty: "Expert"
---

# A flash-sale postmortem shows the reservation ledger became the single serialization bottleneck and throttled the whole checkout path. Walk through how you'd re-architect to keep correctness (no oversell) while removing the global hot spot, and how you'd prove the new design under load before peak.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Exactly Once via Idempotency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Exactly_Once_via_Idempotency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To address the concurrency bottleneck in a flash-sale system where the reservation ledger acts as a single serialization point, we need to re-architect using distributed locks and a microservices approach. This will ensure correctness by preventing overselling while avoiding the global hot spot issue. We’ll use etcd for distributed locking and implement a queuing mechanism with RabbitMQ to process reservations asynchronously. Before going live, thorough load testing is critical to prove that our new design can handle peak traffic without degradation.

#### Triaging the Question

The key challenge here is ensuring correctness in a highly concurrent environment where multiple users attempt to make transactions simultaneously. The current single-threaded reservation ledger acts as a bottleneck and limits overall throughput during flash sales. The goal is to maintain atomicity, consistency, isolation, and durability (ACID) properties while scaling the system horizontally.

#### Assumptions
Single-Threaded Reservation Ledger: Currently, all reservations are processed sequentially through this single point.
Global Hot Spot: High contention occurs at the reservation ledger during flash sales.
Flash Sale Volume: Expected to spike dramatically during peak times (e.g., 50,000+ concurrent users).
Correctness Requirement: Must prevent overselling and ensure data consistency.

#### First-Principles Solution

Distributed Locks with etcd:

Use a distributed key-value store like etcd to manage locks for each item during the flash sale.
Each reservation request will first attempt to acquire a lock on the relevant item before proceeding with any transaction.
If a user tries to reserve an item when another user has already locked it, their request will block until the lock is released.

Microservices Architecture:

Break down the system into smaller microservices responsible for different parts of the checkout path (e.g., inventory management, payment processing).
Each microservice can operate independently and communicate via well-defined APIs.
Use a message broker like RabbitMQ to decouple services so that they can scale more easily.

Asynchronous Processing with RabbitMQ:

Introduce an asynchronous queuing mechanism where reservation requests are placed in a queue before being processed.
This allows the system to handle bursts of traffic without blocking and ensures fair processing of all requests.
Implement a worker service that pulls reservations from the queue, processes them, and updates the ledger.

Load Testing:

Develop comprehensive load testing scenarios using tools like JMeter or Gatling to simulate peak conditions.
Test each component separately (microservices, RabbitMQ, etcd) as well as end-to-end workflows.
Analyze metrics such as latency, throughput, and resource utilization under various loads.

#### Closing Summary

By implementing a distributed locking mechanism with etcd and adopting a microservices architecture that leverages asynchronous processing via RabbitMQ, we can effectively manage concurrency during flash sales without compromising on correctness. Load testing is crucial to validate our design against real-world scenarios, ensuring the system performs well even under heavy load.

#### Common Mistakes/Follow-Ups
Ignoring Local Locks: Using local locks within a single instance instead of distributed ones can lead to race conditions.
Overcomplicating the Solution: A simpler approach using message queues might be sufficient if the spike in traffic is manageable with current infrastructure.
Failing Load Testing: Skipping or inadequately conducting load testing can result in surprises during actual sales events.

**Comparison:**

```
Component	Functionality
etcd	Manage distributed locks
RabbitMQ	Asynchronous message queuing
Microservices	Handle reservation logic asynchronously
```

**Code:**

```python
# Example Python code for RabbitMQ producer and consumer
import pika

def send_to_queue(item_id):
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='reservations')

    message = {'item_id': item_id, 'user_id': user_id}
    channel.basic_publish(exchange='', routing_key='reservations', body=str(message))
    print(f" [x] Sent {message}")
    connection.close()

def process_queue():
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()
    channel.queue_declare(queue='reservations')

    def callback(ch, method, properties, body):
        reservation_data = json.loads(body)
        item_id = reservation_data['item_id']
        user_id = reservation_data['user_id']

        # Process the reservation
        process_reservation(item_id, user_id)

    channel.basic_consume(queue='reservations', on_message_callback=callback, auto_ack=True)
    print(' [*] Waiting for messages. To exit press CTRL+C')
    channel.start_consuming()
```

**Example input / output:**

```text
- **Input**: User makes a reservation request for item ID `101` with user ID `user_123`.
- **Output**: Reservation request is placed in the RabbitMQ queue and processed by a worker service.
```

**Data:**

```json
{
  "item_id": 101,
  "user_id": "user_123"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Makes ReservationRequest
6. Is Lock Available?
7. Acquire Lock
8. Process Reservation
9. Lock Timeout?
10. Release Lock and Retry Later
11. Cancel Reservation
12. Wait for Lock to BecomeAvailable
13. Update Ledger
14. Release Lock

