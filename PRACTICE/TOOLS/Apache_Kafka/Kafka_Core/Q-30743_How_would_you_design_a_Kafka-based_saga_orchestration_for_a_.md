---
type: "practice"
difficulty: "Expert"
---

# How would you design a Kafka-based saga orchestration for a multi-step booking flow, ensuring compensating actions run if a later step fails?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent and Transactional Producers](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Idempotent_and_Transactional_Producers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design a robust Kafka-based saga orchestration for handling a multi-step booking process, with compensating actions in place to ensure that any failed steps can be rolled back gracefully. This involves setting up producers and consumers, managing transactions, and ensuring the correct flow of messages between steps.

#### Triage

The question is asking us to design an end-to-end system using Kafka for orchestrating a multi-step booking process. We need to account for compensating actions in case any step fails, which means we must handle failures gracefully by rolling back to previous states. The interviewer will likely be probing our understanding of Kafka's transactional capabilities, state management, and error handling mechanisms.

#### Assumptions
Each step in the booking flow is a service that publishes events or commands via Kafka topics.
There are compensating actions associated with each step that can roll back changes made by previous steps.
We need to ensure atomicity of operations despite potential failures at any point during the process.
The system should be resilient and capable of handling partial failures.
Step-by-Step Solution
Define Service Steps: Break down the booking flow into discrete steps, such as reservation, payment, confirmation, etc.
Event-Driven Architecture: Each step will publish events to Kafka topics. For example:
Reservation step publishes ReservationCreated event.
Payment step publishes PaymentSuccessful or PaymentFailed event.
Saga Orchestrator: Implement a saga orchestrator that listens for these events and manages the state of each booking process. The orchestrator will coordinate the steps by sending appropriate commands to subsequent services.
Compensating Actions: For each step, define compensating actions (e.g., if payment fails, cancel reservation) and ensure they are executed atomically with the failing step.

#### Implementation
Details
Kafka Topics and Producers:
Create Kafka topics for each service step to publish events.
Use Kafka producers to send commands from one service to another.
Saga Orchestrator Code (in Python):
PYTHON
from kafka import KafkaProducer, KafkaConsumer, TopicPartition

# Initialize Kafka producer and consumer
producer = KafkaProducer(bootstrap_servers='localhost:9092')

#### cons
umer = KafkaConsumer(
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest'
)

def process_event(event):
    # Process the event based on its type
    if event['type'] == 'ReservationCreated':
        handle_reservation_created(event)
    elif event['type'] == 'PaymentSuccessful':
        handle_payment_successful(event)
    else:
        # Handle other types of events or errors
        pass

def handle_reservation_created(event):
    # Create reservation logic here
    print(f"Handling ReservationCreated: {event}")

    # Publish command to PaymentService topic
    producer.send('PaymentService', value=b'PAYMENT_REQUEST')

def handle_payment_successful(event):
    # Payment logic and confirm booking
    print(f"Handling PaymentSuccessful: {event}")

    # Publish confirmation to BookingConfirmation topic
    producer.send('BookingConfirmation', value=b'BILLING_CONFIRMED')

# Consumer loop to listen for events
for event in consumer:
    process_event(event.json())
Compensating Actions (using a compensating action example):
If the payment step fails, the orchestrator should send a command to cancel the reservation.
PYTHON
def handle_payment_failure():
    # Send command to ReservationService to cancel booking
    producer.send('ReservationService', value=b'RESERVATION_CANCELLATION_REQUEST')

#### Closing Summary
and Common Mistakes/Follow-Ups

In conclusion, designing a Kafka-based saga orchestration involves carefully defining the steps of the booking process, leveraging Kafka's transactional capabilities for managing state changes, and ensuring that compensating actions are in place to handle failures gracefully. Key challenges include robust error handling, managing state consistency across multiple services, and ensuring high availability.

#### Common mistakes
include not properly managing transaction boundaries, neglecting edge cases (e.g., concurrent failures), or failing to test the entire flow thoroughly. Follow-up questions might explore how you would handle retries, logging, and monitoring of such a system.

**Comparison:**

```
Service Step	Event Type	Action
Reservation	ReservationCreated	Publish to PaymentService
Payment	PaymentSuccessful	Confirm Booking
Confirmation	-	N/A
Service Step	Compensating Action
Payment	Cancel Reservation (if payment fails)
```

**Code:**

```python
# Example Kafka producer and consumer setup in Python using Kafka library
from kafka import KafkaProducer, KafkaConsumer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
consumer = KafkaConsumer(
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest'
)

def process_event(event):
    if event['type'] == 'ReservationCreated':
        handle_reservation_created(event)
    elif event['type'] == 'PaymentSuccessful':
        handle_payment_successful(event)
    else:
        pass

def handle_reservation_created(event):
    print(f"Handling ReservationCreated: {event}")
    producer.send('PaymentService', value=b'PAYMENT_REQUEST')

def handle_payment_successful(event):
    print(f"Handling PaymentSuccessful: {event}")
    producer.send('BookingConfirmation', value=b'BILLING_CONFIRMED')

for event in consumer:
    process_event(event.json())
```

**Example input / output:**

```text
Input:
- Reservation service creates a reservation and publishes `ReservationCreated` event.

Output:
1. Kafka consumer picks up the `ReservationCreated` event.
2. Saga orchestrator sends `PAYMENT_REQUEST` to PaymentService topic.
3. Payment is successful, so Saga orchestrator processes `PaymentSuccessful` event and confirms booking.
```

**Data:**

```json
{
  "events": [
    {"type": "ReservationCreated", "data": {}},
    {"type": "PaymentSuccessful", "data": {}}
  ]
}
```

**Flow:**

1. Reservation Created
2. Payment Requested
3. Payment Successful
4. Booking Confirmed

