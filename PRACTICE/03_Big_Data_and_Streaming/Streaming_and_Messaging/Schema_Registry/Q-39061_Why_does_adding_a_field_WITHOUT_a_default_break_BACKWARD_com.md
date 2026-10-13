---
type: "practice"
difficulty: "Deep"
---

# Why does adding a field WITHOUT a default break BACKWARD compatibility but adding one WITH a default not — walk through what the consumer's reader schema does at deserialization?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Phased Schema Rollouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Phased_Schema_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why adding a field without a default breaks backward compatibility while adding one with a default does not, we need to delve into how Schema Registry works and its role in ensuring data integrity during schema evolution.

Schema Registry is a central service that manages the versioning of schemas for your applications. When you define a new schema, it can be registered and stored in this registry. Consumers of the data rely on these registered schemas to correctly deserialize incoming messages. If the schema changes (e.g., by adding or removing fields), consumers need to update their reader schemas to handle the changes.

When you add a field without a default value, the deserialization process encounters an issue because there is no way for the consumer’s reader schema to know what value this new field should have. This leads to potential errors and inconsistent data handling during deserialization.

On the other hand, adding a field with a default value allows the deserializer to provide a sensible fallback in case the producer's schema has evolved but the consumer hasn't yet updated its schema. The reader can use the default value provided by the schema to ensure that the data can still be consumed without breaking the application logic.

#### Triaging the Question

The question is probing:

Understanding of how schema evolution impacts deserialization.
Knowledge of default values and their role in ensuring backward compatibility.
Awareness of potential issues during deserialization when fields are added or removed from schemas.

#### First-Principles Solution

Let's break down the process step-by-step:

Schema Evolution: When a new schema is registered, it replaces the old one. Consumers need to update their reader schemas to handle the new structure.
Deserialization Process: During deserialization, the reader schema must match the producer’s schema exactly for backward compatibility. If there are differences (e.g., added fields), the consumer needs to account for these changes.
Adding a Field Without Default:
Producer Schema: { "name": string }

#### Cons
umer Schema: { "name": string }
New Producer Schema: { "name": string, "age": int } (without default)
Deserialization Issue: The consumer cannot determine the value of age, leading to an error or missing data.
Adding a Field With Default:
Producer Schema: { "name": string, "age": 0 }

#### Cons
umer Schema: { "name": string } (not updated)
Deserialization Process: The consumer can safely use the default value 0 for age, ensuring data integrity and avoiding deserialization errors.

#### Closing Summary

Adding a field without a default breaks backward compatibility because it forces consumers to update their schemas. Adding a field with a default ensures that consumers can handle the new schema gracefully, using the default values where necessary. This approach minimizes disruptions and allows for smooth evolution of schemas over time.

#### Common Mistakes
and Follow-Ups
Mistake: Ignoring potential issues when adding fields without defaults.

#### Follow-Up
Discussing how to handle complex schema changes (e.g., removing fields, renaming fields) and their impact on backward compatibility.

