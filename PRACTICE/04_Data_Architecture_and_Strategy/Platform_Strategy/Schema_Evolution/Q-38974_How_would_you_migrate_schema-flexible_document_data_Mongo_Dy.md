---
type: "practice"
difficulty: "Expert"
---

# How would you migrate schema-flexible document data (Mongo/DynamoDB) that evolved organically for years into a governed, versioned model? Talk about lazy vs eager migration and the read-path compatibility layer.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrations with Read Your Writes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In migrating an unstructured or loosely governed schema from a document database like MongoDB or DynamoDB to a more structured, version-controlled model, we need a systematic approach that balances minimal disruption with thoroughness. The key challenges include ensuring data integrity and compatibility during the migration process, especially when dealing with legacy data that has evolved organically over time. This involves strategic planning around lazy vs eager migrations and implementing a read-path compatibility layer to support seamless transitions.

First, let's define our objectives:

Data Integrity: Ensure that all existing data remains intact.
Compatibility: Maintain functional continuity during the transition period.
Version Control: Implement a strategy for managing schema changes over time without disrupting ongoing operations.
Operational Transparency: Minimize downtime and ensure smooth migration.

#### Triage
The Problem and What We're Probing

The core of this question revolves around transitioning an unstructured or loosely governed database into a more controlled, versioned model. The interviewer is probing your understanding of:

The nuances of schema evolution in document databases.
Techniques for managing data migration to avoid loss or corruption.
Strategies for maintaining operational stability during the transition.

#### Assumptions
Data Volume: Large volume of existing data that has evolved organically over years.
Read-Write Load: Significant read and write operations on the database, necessitating minimal downtime.
Schema Complexity: Diverse schema structures with potential inconsistencies or anomalies.
Lazy vs Eager Migration
Lazy Migration

Definition: In a lazy migration approach, data is migrated as needed based on specific queries or application requirements. This reduces initial overhead and allows for incremental changes.

#### Pros
Minimizes downtime by only migrating necessary parts of the schema.
Reduces resource usage during the transition phase.
Easier to implement and manage in production environments.
Eager Migration

Definition: In an eager migration, all data is migrated before it becomes available for reading or writing. This ensures a consistent schema across the entire dataset but requires more preparation time.

#### Pros
Ensures that the final schema is fully aligned with the new governance rules from day one.
Simplifies maintenance and reduces complexity during read operations.

#### Trade-offs
Lazy Migration can lead to inconsistent queries, as data may be in different states depending on when it was last migrated.
Eager Migration requires more upfront effort but provides a cleaner, more consistent state across the database.
Read-Path Compatibility Layer

A compatibility layer is crucial for maintaining read-path integrity during migration. This involves creating a facade or adapter that translates queries from the old schema to the new one.

Designing the Compatibility Layer
Schema Mapping: Define mappings between the old and new schemas.
Proxy Logic: Implement logic in the compatibility layer to dynamically adapt query parameters and data structures.
Fallback Mechanisms: Handle cases where legacy data cannot be directly translated, providing fallbacks or error messages.

#### Example
Workflow

Suppose you have an existing document user with fields like {name: String, age: Number}. The new schema might require additional fields such as email, country, and preferences.

Old Query: db.users.find({ name: "John Doe" })
Compatibility Layer Mapping:
JAVASCRIPT
function translateQuery(query) {

#### cons
t translated = {};
  if (query.name) {
    translated.email = query.name; // Assuming email is derived from name in some way
  }
  return translated;
}

#### Implementation
Example
JAVASCRIPT
// Old Data Structure

#### cons
t oldUserDocument = { _id: ObjectId("507f191e810c19729de860ea"), name: "John Doe", age: 30 };

// New Data Structure

#### cons
t newUserDocument = {
  _id: ObjectId("507f191e810c19729de860ea"),
  email: "john.doe@example.com",
  country: "USA",
  preferences: { theme: "dark", notifications: true }
};

// Compatibility Layer
function migrateUser(oldDoc) {

#### cons
t newUser = {
    _id: oldDoc._id,
    email: deriveEmailFromName(oldDoc.name), // Placeholder for actual logic
    country: determineCountryFromName(oldDoc.name), // Placeholder for actual logic
    preferences: {} // Additional fields can be added here
  };
  return newUser;
}

// Migrate Existing Data

#### cons
t migratedUsers = oldUsers.map(migrateUser);

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key to a successful schema migration in document databases is careful planning, leveraging lazy or eager migrations based on the specific needs of your environment, and implementing robust compatibility layers. Common pitfalls include over-engineering the initial transition, failing to account for edge cases, and not adequately testing the new schema before full deployment.

#### Follow-ups
could involve:

Discussing performance implications.
Exploring automated tools for schema evolution.
Dealing with complex data dependencies or inconsistencies.

**Comparison:**

```
Old Field	New Field
name	email
	country
	preferences
```

**Code:**

```javascript
// Example Code for Schema Mapping
function translateQuery(query) {
  const translated = {};
  if (query.name) {
    translated.email = query.name; // Assuming email is derived from name in some way
  }
  return translated;
}
```

**Example input / output:**

```text
Input:
- Old User Document: { _id: ObjectId("507f191e810c19729de860ea"), name: "John Doe", age: 30 }

Output:
- Migrated User Document: {
    "_id": ObjectId("507f191e810c19729de860ea"),
    "email": "john.doe@example.com",
    "country": "USA",
    "preferences": {}
  }
```

**Data:**

```json
{
  "oldUserDocument": { "_id": ObjectId("507f191e810c19729de860ea"), "name": "John Doe", "age": 30 },
  "newUserDocument": {
    "_id": ObjectId("507f191e810c19729de860ea"),
    "email": "john.doe@example.com",
    "country": "USA",
    "preferences": {}
  }
}
```

**Flow:**

1. Yes
2. No
3. Query
4. Is Lazy Migration?
5. Migrate Incrementally
6. Eagerly Migrate All Data
7. Translate Queries Dynamically
8. Ensure Full Schema Alignment
9. Deploy Compatibility Layer

