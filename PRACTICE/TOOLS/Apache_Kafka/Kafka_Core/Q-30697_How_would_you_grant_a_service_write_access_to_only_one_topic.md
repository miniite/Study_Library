---
type: "practice"
difficulty: "Deep"
---

# How would you grant a service write access to only one topic prefix using ACLs, and what resource patterns exist?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Topic Design for Re-sync](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Topic_Design_for_Re-sync.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To grant a service write access to only one topic prefix in Apache Kafka using Access Control Lists (ACLs), we need to understand the different types of resources and patterns available in Kafka's security model. The key idea is to leverage these patterns to ensure that services are granted precise permissions, without affecting other topics.

#### Triage

The question asks for a detailed explanation on how to configure ACLs to provide write access to only one topic prefix. This involves understanding the different resource types and pattern matching in Kafka's security model. The interviewer is probing your knowledge of Kafka's security architecture, specifically ACLs and their configuration options.

Step-by-Step Solution
Understand Kafka's Security Model: Kafka supports three levels of access control: cluster level, topic level, and operation level.
Identify the Resources Involved: In this case, we are dealing with a specific topic prefix and the write operation.
Choose the Correct Resource Type: The relevant resource type here is the "topic pattern."
Configure the ACLs:
Define a topic pattern that matches your desired topic prefix.
Grant write permissions to the service using this pattern.
Detailed Steps
Define Topic Pattern: A topic pattern in Kafka can be defined using a wildcard character *. For example, if you want to allow writes on topics starting with orders-, the pattern would be orders-*.
Create ACLs: Use the Kafka command-line tools or API to create the necessary ACLs.
BASH
kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 \
  --add --allow-principal User:service-name --operation Write --pattern-type prefix \
  --topic orders-* | true
Verify Configuration: Ensure that the service can write to topics under orders- but not to other topics.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, configuring ACLs for a specific topic prefix involves defining an appropriate pattern and using Kafka’s command-line tools or API to apply the necessary permissions. Key considerations include:

Precision in Patterns: Ensure the patterns are correctly defined to avoid unintended access.
Test Permissions: Verify that the service can perform the desired operations without affecting other topics.

#### Common mistakes
often involve overly broad or overly restrictive patterns, which can lead to either security vulnerabilities or operational issues. Follow-up questions might explore alternative configurations or discuss best practices for securing Kafka clusters.

**Comparison:**

```
ACL	Principal	Operation	Pattern Type	Topic
Add	User:service-name	Write	Prefix	orders-*
```

**Code:**

```bash
# Example ACL command using kafka-acls.sh script
kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 \
  --add --allow-principal User:service-name --operation Write --pattern-type prefix \
  --topic orders- | true
```

**Example input / output:**

```text
Input:
ACL Command: kafka-acls.sh --authorizer-properties zookeeper.connect=localhost:2181 \
--add --allow-principal User:service-name --operation Write --pattern-type prefix \
--topic orders-

Output:
ACLs applied successfully.
```

**Data:**

```json
{
  "topics": ["orders-001", "orders-002", "metrics"]
}
```

**Flow:**

1. Yes
2. No
3. Define Topic Pattern
4. Is the pattern correct?
5. Create ACLs
6. Verify Service Permissions

