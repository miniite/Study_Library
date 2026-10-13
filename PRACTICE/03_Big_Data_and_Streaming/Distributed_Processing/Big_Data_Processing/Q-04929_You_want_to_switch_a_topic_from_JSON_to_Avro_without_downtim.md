---
type: "practice"
difficulty: "Deep"
---

# You want to switch a topic from JSON to Avro without downtime; walk me through a dual-write or dual-read migration and how you'd verify no records are lost.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Topic and Schema Design for Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a Kafka topic from JSON to Avro format without downtime, we need to implement a dual-write strategy that ensures both formats coexist temporarily until the entire data processing pipeline is fully migrated to Avro. This involves setting up a new Avro producer and consumer alongside the existing one for a transition period.

#### Triaging the Question

The key aspect here is ensuring zero loss of records during the migration, which requires careful planning and execution. We will cover the steps involved in dual-write, verify no data loss through checksums or duplicate keys, and discuss potential pitfalls and best practices.

#### Assumptions
Kafka brokers are healthy and available.
Both JSON and Avro producers/consumers are functioning correctly.
The topic structure remains consistent between both formats (e.g., same key-value pairs).
Data integrity is crucial, so any lost records will be a significant concern.
Dual-Write Migration Strategy

Initial Setup:

Create the new Avro topic on Kafka with identical settings to the existing JSON topic.
Ensure all producers are configured to write to both topics simultaneously (dual-write).

#### Cons
umer Setup:

Develop a consumer that reads from both JSON and Avro topics concurrently.
Implement logic to detect any discrepancies in data between the two formats.

Data Verification:

Use checksums or unique IDs to verify record integrity across the dual-consumers.
Store these verification results for post-migration analysis.

Monitoring & Logging:

Set up monitoring tools to track consumption rates, latency, and any anomalies.
Maintain detailed logs of all consumer activities during this phase.

Graceful Downtime Transition:

Once the data migration is complete, gradually shut down the JSON producer and consumer, ensuring no new records are written in the old format.
Validate that the Avro consumer can handle the transition without issues.

Final Cleanup:

Delete the old JSON topic if no further use of it is expected.
Step-by-Step Solution

Schema Conversion & Validation:

Start
Convert JSON schema to Avro
Validate Avro schema using atool like `avro-tools`
Create new Avro topic onKafka

Dual-Write Implementation:

Configure the existing producer to write data to both the JSON and Avro topics.
PYTHON
from kafka import KafkaProducer

# Example using Python Kafka library
json_producer = KafkaProducer(bootstrap_servers='kafka-broker1,kafka-broker2')
avro_producer = KafkaProducer(bootstrap_servers='kafka-broker1,kafka-broker2', value_serializer=avro_serializer)

def send_data(topic, message):
    # Send JSON data to the old topic
    json_producer.send(topic, key=message['key'], value=str(message))

    # Serialize and send Avro data to the new topic
    avro_producer.send('new_avro_topic', key=message['key'], value=avro_message)

# Example message (assuming JSON)

#### example
_json = {
    'key': 1,
    'data': {'name': 'John Doe'}
}
send_data('old_json_topic', example_json)

Dual-Read Consumer Setup:

Develop a consumer that reads from both topics.
PYTHON
from kafka import KafkaConsumer

def consume_both_topics():
    json_consumer = KafkaConsumer('old_json_topic', bootstrap_servers='kafka-broker1,kafka-broker2')
    avro_consumer = KafkaConsumer('new_avro_topic', bootstrap_servers='kafka-broker1,kafka-broker2')

    for message in json_consumer:
        print(f"JSON Consumer: {message.value.decode()}")

    for message in avro_consumer:
        print(f"Avro Consumer: {str(message.value)}")

Data Integrity Verification:

Implement logic to check for inconsistencies.
PYTHON
def verify_data_integrity(json_msg, avro_msg):
    if json_msg.key != avro_msg.key or json_msg.value != str(avro_msg.value):
        print("Mismatch detected!")

Monitoring & Logging:

Use monitoring tools like Prometheus and Grafana to track the consumer's health.
PYTHON
from prometheus_client import start_http_server, Gauge

def monitor_consumers():
    json_gauge = Gauge('json_consumer_count', 'Number of JSON messages consumed')
    avro_gauge = Gauge('avro_consumer_count', 'Number of Avro messages consumed')

    for message in json_consumer:
        json_gauge.inc()

    for message in avro_consumer:
        avro_gauge.inc()

Graceful Shutdown:

Gradually transition by stopping the JSON consumer and producer.
PYTHON
def graceful_shutdown():
    # Wait for a grace period (e.g., 1 hour)
    time.sleep(3600)

    # Stop producers/consumers
    json_producer.close()
    avro_producer.close()
    json_consumer.close()
    avro_consumer.close()

    print("Graceful shutdown completed.")

#### Closing Summary
& Common Mistakes/Follow-Ups

By implementing a dual-write strategy, we ensure that no data is lost during the migration process. The key steps involve setting up both JSON and Avro producers/consumers, verifying data integrity through checksums or unique IDs, monitoring the transition phase, and ensuring a smooth shutdown.

#### Common mistakes
include:

Failing to validate schema conversions.
Not properly handling edge cases like duplicate keys or null values.
Overlooking latency issues that could affect real-time processing.
Neglecting thorough logging and monitoring during critical phases.

#### Follow-ups
might include discussing how to handle large volumes of data, ensuring compatibility with downstream systems, and strategies for dealing with potential failures during the transition period.

