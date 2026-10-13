---
type: "practice"
difficulty: "Deep"
---

# What governance would you put around a continuously-updated RAG corpus fed by user uploads, given the poisoning and prompt-injection risk from ingested content?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ RAG Corpus Governance](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/RAG_Corpus_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with an AI system that processes real-time updates to its knowledge base via user uploads. The primary risks are data poisoning (maliciously crafted inputs corrupting the model) and prompt injection (adversarial prompts manipulating the model's behavior). To address these, a comprehensive governance strategy is essential.

First, let’s define our core objectives: ensuring data integrity, preventing harmful content from being processed, maintaining transparency, and enabling continuous improvement while mitigating risks. Now, we'll break down the solution step by step.

**Comparison:**

```
Step	Action	Tool
Ingestion	Collect user-uploaded data.	API Gateway
Cleaning	Remove irrelevant or malformed data.	Data Cleansing Tools
Validation	Check content against predefined safety and factuality rules.	Rule-Based Systems
Normalization	Standardize the format for consistency.	NLP Libraries
Storage	Save processed data in a secure, scalable repository.	Distributed File System
```

**Code:**

```
MERMAID
Copy
graph LR
    A[Data Ingestion] --> B(Cleaning)
    B --> C[Validation]
    C --> D[Normalization]
    D --> E[Storage]
    A --> F[Moderation]
```

**Example input / output:**

```
Input: User uploads a CSV containing text data.
Output: Cleaned, validated, and stored in the system.

Example:

JSON
Copy
{
  "raw_data": {
    "timestamp": "2023-10-04T15:37:06Z",
    "user_id": "u12345",
    "content": "This is a test upload that may contain harmful content."
  },
  "processed_data": {
    "cleaned_content": "This is a test upload that may contain harmful content.",
    "validated": false,
    "normalized_steps": ["Tokenization", "Stemming"],
    "storage_status": "Queued for storage"
  }
}
```

**Data:**

```yaml
data:
  - ingestion_source: "User Uploads"
  - validation_rules: ["Safety", "Factuality"]
  - normalization_steps: ["Tokenization", "Stemming", "Lemmatization"]
  - storage: {type: "Distributed File System", capacity: "TB+"}
```

**Flow:**

1. Ingestion
2. Cleaning
3. Validation
4. Normalization
5. Storage
6. Moderation
7. Audit Logs

