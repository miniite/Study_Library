---
type: "practice"
difficulty: "Expert"
---

# How would you achieve exactly-once processing from Kafka into a downstream database?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Exactly Once Beyond Kafka](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_Beyond_Kafka/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve exactly-once processing from Kafka to a downstream database, we need to ensure that each message is processed exactly once, without any duplication or loss. This involves leveraging Apache Kafka's idempotent producer features and integrating with a database that supports atomic transactions or implementing compensating operations if the database does not support ACID transactions natively.

The key steps involve:

Configuring the Kafka producer to be idempotent.
Ensuring the downstream database either supports transactional semantics (ACID) or can handle retries and compensations appropriately.
Handling failures gracefully, ensuring that duplicates are avoided and operations are idempotent where possible.

Let's break down these steps in detail:

#### Triaging the Question

The question is probing our understanding of distributed systems and how to ensure data consistency across multiple layers of processing (Kafka -> Database). Key considerations include:

Idempotency: Ensuring that messages can be retried without causing side effects.
Transactional Semantics: Handling transactions in the database to ensure atomicity, consistency, isolation, and durability.

#### Assumptions
We are using a Kafka cluster with idempotent producers configured.
The downstream database supports transactional semantics or we have compensating operations in place.

#### Solution
from First Principles

Idempotent Producer Configuration: Configure the Kafka producer to be idempotent, which means that even if a message is sent multiple times (due to network issues), it will not result in duplicate processing on the consumer side. This can be achieved using Kafka's idempotentProducer feature.

Database Transaction Support:

If the database supports ACID transactions natively (e.g., PostgreSQL, MySQL with InnoDB engine), use transactional semantics to ensure that writes are atomic and consistent.
For databases without native support for transactions or when using NoSQL databases like Cassandra, implement compensating operations. These are actions taken to roll back a previous operation if the initial operation fails.

Handling Failures:

Implement retry logic in the consumer to handle transient failures.
Use dead-letter queues (DLQ) to manage messages that fail multiple times and require manual intervention or analysis.

#### Example
Implementation

Let's consider an example where we use a PostgreSQL database with transactional support:

PYTHON
from kafka import KafkaProducer, KafkaConsumer
import psycopg2

# Configure idempotent producer in Kafka
producer = KafkaProducer(bootstrap_servers=['localhost:9092'],
                         client_id='my-producer',
                         enable_idempotence=True)

# Database connection setup
db_connection = psycopg2.connect(
    user="user",
    password="password",
    host="localhost",
    port="5432",
    database="database_name"
)
cursor = db_connection.cursor()

def process_message(message):
    try:
        # Begin transaction
        db_connection.autocommit = False

        # Process the message (e.g., insert into a table)
        cursor.execute("INSERT INTO my_table (id, value) VALUES (%s, %s)", (message.key(), message.value()))

        # Commit transaction if successful
        db_connection.commit()
    except Exception as e:
        print(f"Error processing message: {e}")
        # Rollback transaction on failure
        db_connection.rollback()
    finally:
        cursor.close()
        db_connection.close()

def consume_from_kafka():

#### cons
umer = KafkaConsumer('my-topic', bootstrap_servers=['localhost:9092'])
    for message in consumer:
        process_message(message)

#### cons
ume_from_kafka()

#### Closing Summary
and Common Mistakes

To ensure exactly-once processing, it is crucial to configure the producer correctly and handle transactions appropriately. Common mistakes include not configuring idempotent producers or neglecting proper transaction management in the database.

Additional Considerations
Dead-Letter Queues (DLQ): For messages that fail multiple times, use a DLQ for further analysis.
Error Handling: Implement robust error handling to manage transient failures gracefully.
Monitoring and Logging: Set up monitoring and logging to detect and address issues early.

**Comparison:**

```
Topic	Partition	Offset	Key	Value
my-topic	0	10	1	"Hello, World!"
```

**Code:**

```python
from kafka import KafkaProducer, KafkaConsumer
import psycopg2

producer = KafkaProducer(bootstrap_servers=['localhost:9092'],
                         client_id='my-producer',
                         enable_idempotence=True)

db_connection = psycopg2.connect(
    user="user",
    password="password",
    host="localhost",
    port="5432",
    database="database_name"
)
cursor = db_connection.cursor()

def process_message(message):
    try:
        db_connection.autocommit = False
        cursor.execute("INSERT INTO my_table (id, value) VALUES (%s, %s)", (message.key(), message.value()))
        db_connection.commit()
    except Exception as e:
        print(f"Error processing message: {e}")
        db_connection.rollback()
    finally:
        cursor.close()
        db_connection.close()

def consume_from_kafka():
    consumer = KafkaConsumer('my-topic', bootstrap_servers=['localhost:9092'])
    for message in consumer:
        process_message(message)

consume_from_kafka()
```

**Example input / output:**

```
Input: Message sent to Kafka topic my-topic.
Output: Message processed and inserted into PostgreSQL table my_table.
Message Key	Message Value
1	"Hello, World!"

Processing result:

SQL
Copy
INSERT INTO my_table (id, value) VALUES (1, 'Hello, World!')
```

**Data:**

```sql
CREATE TABLE my_table (
    id INT PRIMARY KEY,
    value TEXT NOT NULL
);
```

**Flow:**

1. Yes
2. No
3. Message Received
4. Is Database Transactional?
5. Process Message and Commit
6. Compensating Operation
7. Log Success
8. Handle CompensatingOperation
9. Message ProcessedSuccessfully
10. Message Not Processed Dueto Compensating Operation

