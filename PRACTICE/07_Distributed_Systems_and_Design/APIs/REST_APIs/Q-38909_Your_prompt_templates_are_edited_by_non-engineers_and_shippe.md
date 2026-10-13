---
type: "practice"
difficulty: "Deep"
---

# Your prompt templates are edited by non-engineers and shipped without a deploy. How do you design a REST-backed prompt registry with versioning, rollout, and rollback so a bad prompt can be reverted fast and every response records which version served it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Versioning and Deprecation](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Versioning_and_Deprecation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a REST-backed prompt registry that includes features like versioning, rollout, and rollback, we need to ensure that prompts are managed with high reliability and traceability. This involves creating a robust API architecture that supports these functionalities while maintaining ease of use for non-engineers.

#### Triage

The core question is how to build an API system that allows prompt templates to be stored, updated, rolled out, and rolled back efficiently. The interviewer is probing your understanding of REST principles, versioning strategies, deployment workflows, logging, and rollback mechanisms. We will assume that the prompts are text-based configurations used in a natural language processing application.

#### First-Principles Solution
Prompt Registry Model: Define a database schema to store prompt versions.
Version Control API Endpoints:
POST /prompt/register: Register new or update existing prompts.
GET /prompt/version/{id}: Retrieve specific version of a prompt.
Rollout Strategy:
Implement a gradual rollout mechanism using A/B testing with the /prompt/rollout endpoint.
Rollback Mechanism: Maintain historical versions to enable quick rollbacks via the /prompt/rollback endpoint.
Logging and Monitoring: Log all API calls for auditing purposes.
Detailed Solution

Database Schema

Prompt
+id: int
+content: string
+version: int
+created_at: datetime
+updated_at: datetime
VersionHistory
+prompt_id: int
+content: string
+version: int
+created_at: datetime

API Endpoints

PYTHON
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/prompt/register', methods=['POST'])
def register_prompt():
    prompt_data = request.json
    # Validate and persist prompt data to the database
    return jsonify({"status": "success", "message": "Prompt registered successfully"})

@app.route('/prompt/version/<int:id>', methods=['GET'])
def get_prompt_version(id):
    # Retrieve specific version from the database
    return jsonify({"version": 1, "content": "Sample prompt content"})

@app.route('/prompt/rollout', methods=['POST'])
def rollout_prompt():
    # Implement A/B testing logic here
    return jsonify({"status": "success", "message": "Rollout initiated"})

@app.route('/prompt/rollback/<int:version>', methods=['GET'])
def rollback_prompt(version):
    # Retrieve and apply the specified version from history
    return jsonify({"status": "success", "message": f"Rolled back to version {version}"})

Rollout Strategy

Implement A/B testing where a portion of users are served the new prompt, while others continue with the old one.
Monitor user feedback and application performance before fully rolling out.

Rollback Mechanism

Maintain a version history table to store all previous versions of prompts.
On rollback, use the latest stored version from the VersionHistory table.

Logging and Monitoring

Log all API interactions in a centralized logging system.
Monitor system health and prompt performance using tools like Prometheus or Grafana.

#### Closing Summary

In summary, we have designed a REST-backed prompt registry with versioning, rollout, and rollback capabilities to ensure high reliability and traceability. This solution uses a combination of database schema management, API endpoints for registration and retrieval, A/B testing for rollouts, version history for rollbacks, and centralized logging for monitoring.

#### Common Mistakes/Follow-Ups
Forgetting to maintain historical versions can lead to loss of critical information.
Insufficient rollback mechanisms might result in prolonged issues if a bad prompt is deployed.
Failing to monitor system health and user feedback can prevent timely identification and resolution of issues.

**Comparison:**

| id | content             | version | created_at          | updated_at          |
|----|---------------------|---------|---------------------|---------------------|
| 1  | Sample prompt       | 1       | 2023-10-01T12:00:00Z| 2023-10-01T12:00:00Z|
| 2  | How can I assist you| 1       | 2023-10-02T12:00:00Z| 2023-10-02T12:00:00Z|

**Code:**

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# Database operations (pseudocode)
db = {}

@app.route('/prompt/register', methods=['POST'])
def register_prompt():
    prompt_data = request.json
    db[prompt_data['id']] = prompt_data
    return jsonify({"status": "success", "message": "Prompt registered successfully"})

@app.route('/prompt/version/<int:id>', methods=['GET'])
def get_prompt_version(id):
    prompt = db.get(id)
    if not prompt:
        return jsonify({"error": "Prompt not found"}), 404
    return jsonify(prompt)

@app.route('/prompt/rollout', methods=['POST'])
def rollout_prompt():
    # Implement A/B testing logic here
    return jsonify({"status": "success", "message": "Rollout initiated"})

@app.route('/prompt/rollback/<int:version>', methods=['GET'])
def rollback_prompt(version):
    # Retrieve and apply the specified version from history
    history = db['history']
    prompt = history.get(version)
    if not prompt:
        return jsonify({"error": "Version not found"}), 404
    db[prompt_data['id']] = prompt
    return jsonify({"status": "success", "message": f"Rolled back to version {version}"})

# Sample data
sample_prompt = {"id": 1, "content": "Sample prompt content"}
db[1] = sample_prompt
```

**Example input / output:**

```text
# Register a new prompt
Input:
{
    "id": 1,
    "content": "Sample prompt content"
}

Output:
{
    "status": "success",
    "message": "Prompt registered successfully"
}

# Retrieve a specific version of a prompt
Input:
GET /prompt/version/1

Output:
{
    "version": 1,
    "content": "Hello user"
}
```

**Data:**

```json
{
    "prompt_registry": {
        "prompts": [
            {"id": 1, "content": "Hello user", "version": 1, "created_at": "2023-10-01T12:00:00Z", "updated_at": "2023-10-01T12:00:00Z"},
            {"id": 2, "content": "How can I assist you today?", "version": 1, "created_at": "2023-10-02T12:00:00Z", "updated_at": "2023-10-02T12:00:00Z"}
        ],
        "history": {
            1: {"id": 1, "content": "Hello user", "version": 2, "created_at": "2023-10-03T12:00:00Z", "updated_at": "2023-10-03T12:00:00Z"}
        }
    }
}
```

**Flow:**

1. Yes
2. No
3. Register Prompt
4. Is Version Specified?
5. Update Existing Prompt
6. Create New Prompt
7. Rollout Prompt
8. Rollout Prompt
9. Rollback if Needed
10. Rollback if Needed
11. Success
12. Success

