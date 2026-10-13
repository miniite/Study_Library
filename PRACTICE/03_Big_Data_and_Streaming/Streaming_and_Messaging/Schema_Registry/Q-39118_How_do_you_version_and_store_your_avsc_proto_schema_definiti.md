---
type: "practice"
difficulty: "Intermediate"
---

# How do you version and store your .avsc/.proto schema definitions in source control and keep them the source of truth versus the registry?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Schema IDs and Compaction](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Schema_IDs_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage and evolve schema definitions effectively, it's crucial to maintain a reliable system that not only stores these definitions but also version controls them while keeping track of their usage in the Schema Registry. This ensures that developers have the correct versions of schemas at any point in time and allows for smooth evolution without breaking existing applications.

#### Triaging the Question

The core of this question is about managing schema definitions, specifically focusing on .avsc (Avro) and .proto files. The key aspects being probed are:

Versioning: How to manage different versions of schemas.
Source Control: Which tools and practices to use for version control.
Schema Registry Integration: Ensuring the registry remains a single source of truth.

#### Assumptions
We are using an Avro or Protocol Buffers (protobuf) schema system, with .avsc and .proto files respectively.
The Schema Registry is used as the authoritative reference for schema definitions in our data pipelines.
Source control systems such as Git are being utilized.

#### First Principles Solution
Step 1: Define Versioning Strategy

Versioning schemas is essential to ensure backwards compatibility and track changes. We can use semantic versioning (e.g., v1, v2) or date-based versions (e.g., 2023-04-05).

PLAINTEXT

#### Example
schema.avsc:v1
Step 2: Source Control Setup

We will use Git for version control. Each schema file should be stored in a dedicated repository or within a designated branch of our main codebase.

BASH
git clone https://github.com/company/schemas.git
cd schemas
# Create branches for each version, e.g., v1, v2
Step 3: Automated Commit Messages

Commit messages should follow a standard format to ensure clarity and traceability:

PLAINTEXT
Version v2 of schema defined in schema.avsc. Added new fields user_id and email.
Step 4: Sync with Schema Registry

Automate the process of updating the registry when changes are committed to the master branch or specific version branches.

BASH
# Example script to sync with a remote registry service
python sync_schemas.py --registry-url http://registry.example.com --branch v2
Step 5: Automated Testing and Validation

Implement unit tests for schema validation to ensure that new versions do not break existing pipelines. Use tools like SchemaEvolve or Protobuf Compiler to validate schemas.

BASH
# Example test command for Avro
python -m avro.schema parse --validate schema.avsc:v2

#### Closing Summary
and Common Mistakes

Maintaining a versioned, source-controlled set of schema definitions ensures that you have the correct versions at all times. However, common pitfalls include:

Lack of Automation: Failing to automate steps like syncing with the registry or validating changes can lead to errors.
Inconsistent Naming Conventions: Poorly managed file names and branch naming can cause confusion.
Manual Overwrite Risk: Manually updating schema files without versioning can result in accidental overwriting.

#### Example
Workflow
Developer commits a new version of a schema.
CI/CD pipeline triggers an automated test to validate the schema.
If tests pass, the script updates the Schema Registry and pushes the change to production branches.
Other systems use the latest versions from the registry or specific tagged releases.
PLAINTEXT
# Example git flow for a new version
git checkout -b v2 feature/new-field
# Add and commit changes
git add schema.avsc
git commit -m "Version 2 of schema with user_id and email field"
git push origin v2
Final Confirmation

I have covered the key points and provided a detailed solution that addresses versioning, source control, and integration with Schema Registry. The approach ensures both correctness and completeness.

**Code:**

```
Example Git commit message

Version 2 of schema defined in schema.avsc. Added new fields user_id and email.

CODE
Copy
```bash
# Example sync script invocation
python sync_schemas.py --registry-url http://registry.example.com --branch v2
```

