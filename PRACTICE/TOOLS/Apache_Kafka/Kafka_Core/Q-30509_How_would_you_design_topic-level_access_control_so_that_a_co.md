---
type: "practice"
difficulty: "Deep"
---

# How would you design topic-level access control so that a compromised producer credential can write to at most one topic and cannot read anything?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Topic Design for Re-sync](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Topic_Design_for_Re-sync.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design topic-level access control in Kafka such that a compromised producer credential writes to at most one topic and does not read from any, we need to leverage Kafka's security mechanisms along with some custom logic. The key challenge is ensuring strong isolation between producers and topics while maintaining flexibility in the number of topics they can write to.

#### Triage

The question is probing your understanding of Kafka’s security features, specifically ACLs (Access Control Lists), topic partitioning, and possibly custom middleware or policy enforcement points. We need to ensure that even if a producer credential is compromised, it should be restricted to writing to only one topic and not allowed any read access.

#### First-Principles Solution
Kafka Security: Enable Kafka security by using SASL (Simplified Authentication and Security Layer) or SSL/TLS for authentication and authorization.
ACLs Setup: Set up ACLs on topics to define which producers can write to them, ensuring that each producer is granted access only to the specific topic(s) they need.
Custom Logic Enforcement: Implement a custom logic layer or use Kafka Connect with appropriate transformations or filtering rules to enforce stricter control over writes and reads.
Step-by-Step Solution

Enable Security:

Enable SASL/PLAIN or other authentication mechanisms in Kafka broker configurations.
Configure SSL/TLS for secure communication between producers and brokers if needed.

Topic-Specific ACLs:

Create an ACL for each topic to specify the producer credentials that can write to it.
Use Kafka's kafka-acls command or REST API to set these ACLs. For example, you might allow a specific producer group to have Producer permission on one topic but not on any other.

Custom Middleware:

Implement a custom middleware (e.g., using Apache NiFi or Kafka Streams) that enforces strict access control.
This middleware can intercept the messages before they reach the Kafka cluster and perform additional checks such as ensuring the message is sent to only one topic.

Audit Logs:

Enable audit logging for both producers and brokers to track any unauthorized attempts and monitor compliance with ACLs.
Use tools like kafka-audit or third-party monitoring tools to keep an eye on access patterns.

#### Closing Summary

By combining Kafka’s built-in security features with custom middleware, we can ensure that even if a producer credential is compromised, it will be restricted to writing only one topic and not have any read capabilities. This setup requires careful configuration of ACLs and the implementation of robust monitoring mechanisms.

#### Common Mistakes/Follow-Ups
Mistake: Failing to implement custom logic or middleware, leading to overly permissive access control.

#### Follow-Up
Discussing how to handle edge cases such as multiple producers sharing a single credential, or scenarios where dynamic topic creation needs to be supported.

**Comparison:**

```
ACL Operation	Host	Principal	Type
Allow	*	valid-producer-credentials	Producer
Topic	Allowed Producers
my-topic	[valid-producer-credentials]
```

**Code:**

```python
# Example Python code for setting ACLs using Kafka REST Proxy API
import requests

def set_topic_acls(topic, producer_credentials):
    url = "http://localhost:8082/topics/{topic}/acl".format(topic=topic)
    data = {
        "operation": "allow",
        "host": "*",
        "principal": producer_credentials,
        "type": "producer"
    }
    response = requests.post(url, json=data)
    if response.status_code != 200:
        print("Failed to set ACLs: ", response.text)

set_topic_acls("my-topic", "valid-producer-credentials")
```

**Example input / output:**

```text
Input:
- Topic: my-topic
- Producer Credentials: valid-producer-credentials

Output:
ACL successfully set for topic 'my-topic' with producer credentials 'valid-producer-credentials'.
```

**Data:**

```json
{
  "topic": "my-topic",
  "producer_credentials": "valid-producer-credentials"
}
```

**Flow:**

1. No
2. Yes
3. Producer Credentials
4. Is Compromised?
5. ACLs Enforced
6. Middleware Enforces Rules
7. Write to Topic Only
8. No Read Access

