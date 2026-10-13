---
type: "practice"
difficulty: "Expert"
---

# How do you architect real-time streaming for security and compliance: encryption in transit and at rest in the log, PII tokenization, access control per topic, and immutable audit — without crippling throughput or reprocessing?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary:

Real-time streaming architectures for security and compliance need to handle encryption in transit and at rest, PII tokenization, access control per topic, and immutable audit logs without impacting throughput or reprocessing significantly. This can be achieved through a layered approach using Kafka, AWS Kinesis, or similar systems, combined with robust encryption mechanisms, tokenization strategies, and fine-grained access controls.

#### Triaging the Question

The question is asking how to build a secure and compliant real-time streaming system that ensures data confidentiality (encryption), integrity (PII tokenization), security of access (topic-level access control), and auditability (immutable logs) without compromising performance. The core aspects being probed include encryption practices, PII handling techniques, authorization mechanisms, and logging strategies.

#### Assumptions
We are working in a cloud environment with tools like AWS Kinesis or Apache Kafka.
The system is expected to handle high-throughput data processing.
Compliance requirements dictate specific security measures (e.g., GDPR for PII).
There's a need for fine-grained access control based on topics.

#### First-Principles Solution
Infrastructure Selection: Choose a robust streaming platform like AWS Kinesis or Apache Kafka, which supports advanced features such as encryption and fine-grained access controls.
Encryption in Transit:
Use TLS/SSL to encrypt data between producers and brokers. This can be enabled by default on most modern streaming platforms.
Implement mutual TLS authentication for additional security layers.
Encryption at Rest:
Encrypt logs using encryption-at-rest mechanisms provided by your chosen platform (e.g., AWS KMS, Kafka's built-in KMS support).
Ensure that keys are managed securely and rotated regularly to minimize risk.
PII Tokenization:
Use tokenization libraries like Apache Beam or AWS Kinesis Firehose to replace PII fields with unique tokens before they hit the stream.
Store tokens separately, ensuring that they can be decrypted only by authorized personnel.
Access Control Per Topic:
Configure topic-level access control using ACLs (Access Control Lists) provided by your streaming platform.
Implement role-based access controls to ensure that only relevant users have permission to read/write specific topics.
Immutable Audit Logs:
Use append-only logs or journals for audit trails to maintain immutability and traceability.
Implement audit logging at the producer and consumer levels to capture all actions and changes.

#### Implementation
Steps:

Provision Streaming Infrastructure:

Provision AWS Kinesis Stream
Configure Encryption in Transit
Enable TLS MutualAuthentication
Set Up Topics for DifferentStreams
Configure ACLs for Each Topic

Tokenization:

Use a tokenization library to replace sensitive data.
PYTHON
from apache_beam.io.aws.kinesis import KinesisWriteFn

class TokenizePIIData(KinesisWriteFn):
    def __init__(self, client):
        self.client = client

    def process(self, element):
        # Tokenize PII fields in the element
        tokenized_element = tokenize(element)
        yield tokenized_element

Encryption and Key Management:

Use AWS KMS or equivalent for key management.
PYTHON
import boto3

client = boto3.client('kms')
encrypted_data = client.encrypt(
    KeyId='your-key-id',
    Plaintext=b'PlainText'
)

Access Control:

Implement ACLs using the Kafka AdminClient API.
PYTHON
from kafka.admin import KafkaAdminClient, NewTopic

admin_client = KafkaAdminClient(bootstrap_servers="localhost:9092")
topics = [NewTopic(name='topic1', num_partitions=3, replication_factor=1)]
admin_client.create_topics(topics=topics)

Audit Logs:

Enable audit logs for producers and consumers.
PYTHON
# Example producer configuration
producer_config = {
    'bootstrap.servers': 'localhost:9092',
    'security.protocol': 'SSL',
    'ssl.ca.location': '/path/to/ca.pem'
}

#### Closing Summary

By leveraging a robust streaming platform with built-in security features, implementing encryption at multiple layers, tokenizing sensitive data, enforcing fine-grained access control, and maintaining immutable audit logs, you can build a secure and compliant real-time streaming system. The key is to balance these requirements without significantly impacting performance.

#### Common Mistakes/Follow-Ups
Ignoring TLS Mutual Authentication: Ensuring mutual authentication is crucial for strong security.
Lack of Key Rotation: Regularly rotating encryption keys reduces the risk of exposure.
Insufficient Tokenization: Failing to properly replace PII fields can lead to compliance breaches.
Overly Permissive Access Controls: Misconfiguring access controls can leave sensitive data exposed.

By addressing these common pitfalls, you ensure that your architecture is both secure and compliant while maintaining high throughput and minimal reprocessing overheads.

**Code:**

```python
from kafka.admin import KafkaAdminClient, NewTopic

# Create a topic with ACLs
admin_client = KafkaAdminClient(bootstrap_servers="localhost:9092")
topics = [NewTopic(name='topic1', num_partitions=3, replication_factor=1)]
admin_client.create_topics(topics=topics)
```

**Flow:**

1. Provision AWS Kinesis Stream
2. Configure Encryption in Transit
3. Enable TLS MutualAuthentication
4. Set Up Topics for DifferentStreams
5. Configure ACLs for Each Topic

