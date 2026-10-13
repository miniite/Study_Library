---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd implement a saga or compensating-transaction pattern in Python across two services where you can't use a distributed transaction.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Global Quotas Across Requests](<../../../../TOOLS/Python/Python_Language/Global_Quotas_Across_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a saga or compensating-transaction pattern in Python across two services without using a distributed transaction, we need to ensure that each service is responsible for its own local transactions and can compensate for failures through explicit messages. This approach ensures loose coupling between services while maintaining the integrity of the overall system.

#### Triaging the Question

The question asks for an implementation of a saga pattern in Python across two services where distributed transactions are not an option. The key here is to handle failures gracefully by using compensating actions, which can be triggered when a service encounters an error and needs to roll back its previous actions.

#### Assumptions
and Key Ideas
Each service should have a well-defined API for initiating actions and handling errors.
Messages will be used to coordinate between services.
We need to ensure idempotency and retries are handled appropriately.
Deadlocks must be avoided by ensuring that compensating actions can be executed in any order.

#### First Principles Solution
Service A: Initiates an action, sends a message to Service B.
Service B: Receives the message, performs its action, sends a confirmation back to Service A.
Service A: Upon receiving the confirmation, it proceeds with its local actions.
If any service fails, it should send a compensating message to revert previous actions.

#### Implementation
Steps
Define a common message format (e.g., JSON).
Implement retry logic and idempotency in both services.
Ensure that messages can be retried without causing side effects.
Use database transactions locally but manage them independently between the two services.

#### Example
Code
PYTHON
import json
from kafka import KafkaProducer, KafkaConsumer
from sqlalchemy.orm import sessionmaker
from sqlalchemy.exc import SQLAlchemyError

# Database setup (example with SQLAlchemy)
engine = create_engine('sqlite:///example.db')
Session = sessionmaker(bind=engine)

# Kafka producer and consumer setup
producer = KafkaProducer(bootstrap_servers='localhost:9092', value_serializer=lambda v: json.dumps(v).encode('utf-8'))

#### cons
umer = KafkaConsumer('saga-topic', bootstrap_servers='localhost:9092', auto_offset_reset='earliest')

def send_message(topic, message):
    producer.send(topic, value=message)

def handle_saga(service_a_action):
    session = Session()
    try:
        # Perform action in Service A
        result = service_a_action(session)

        # Send confirmation to Service B
        send_message('service-b-topic', {'action': 'confirmation'})

        # Commit local transaction
        session.commit()
    except SQLAlchemyError as e:
        # Rollback and retry locally if possible
        session.rollback()
        print(f"Local rollback: {e}")
        handle_saga(service_a_action)  # Retry the action

def service_b_handler(message):
    try:
        # Perform action in Service B
        result = perform_service_b_action(message)

        # Send confirmation back to Service A
        send_message('service-a-topic', {'action': 'confirmation'})

        # Commit local transaction
        session.commit()
    except SQLAlchemyError as e:
        # Rollback and retry locally if possible
        session.rollback()
        print(f"Local rollback: {e}")
        handle_saga(service_b_handler)  # Retry the action

def perform_service_a_action(session):
    try:
        # Example local transaction
        result = some_business_logic(session)
        return result
    except Exception as e:
        raise SQLAlchemyError(e)

def perform_service_b_action(message):
    try:
        # Example local transaction
        result = some_other_business_logic()
        return result
    except Exception as e:
        raise SQLAlchemyError(e)

# Kafka consumer loop
for message in consumer:
    service_b_handler(json.loads(message.value))

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: The key to implementing a saga pattern is to ensure that each service can handle its local transactions independently, and compensate for failures by sending messages back. This requires robust error handling and retry logic.

#### Common Mistakes
Ignoring idempotency and leading to duplicated actions.
Failing to handle deadlocks properly, which can cause infinite loops.
Not using a message broker like Kafka for coordination between services.

#### Follow-Ups
How do you ensure that messages are not lost?
What happens if both services fail at the same time?
Can you explain how back-pressure is handled in this implementation?

