---
type: "practice"
difficulty: "Deep"
---

# You must design a class-level contract-testing seam so a provider service can't ship a breaking API change without a consumer's test failing; how do you model the contract and where does verification run?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Domain Event Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/Domain_Event_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a provider service doesn’t deploy a breaking API change without first verifying it with consumers, we need to design a robust class-level contract-testing seam. This involves defining clear, unambiguous contracts and ensuring that verification runs in a way that prevents accidental or intentional breaks. We'll approach this from the perspective of both a backend SDE-2/3 and an Architect/Principal.

#### Triaging the Question

The question is asking for a design to enforce API contract compliance between provider and consumer services. Specifically, we need:

A robust way to define the service contracts.
Mechanisms to verify these contracts are being followed without breaking changes.
Where and when this verification should occur.
Key Assumptions
Both provider and consumer services will be part of a larger microservices ecosystem.
The contract must cover not only method signatures but also their behavior, including return types and error handling.
Verification must happen before deployment to prevent breaking changes from propagating.
Step-by-Step Solution
1. Define the Contract

We'll define the service contracts using OpenAPI (formerly known as Swagger) or a similar specification language. This will ensure that both provider and consumer services have clear expectations about what is being called, how it behaves, and what responses to expect.

YAML
# Example OpenAPI Specification for a User API
openapi: 3.0.1
info:
  title: User Service
  version: "1.0"
paths:
  /users/{id}:
    get:
      summary: Retrieve a user by ID.
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User data returned.
          content:
            application/json:
              schema:
                type: object
                properties:
                  username: {type: string}
                  email: {type: string}
2. Implement a Contract Verification Mechanism

The verification mechanism should run in the consumer service, acting as a boundary between the provider and the rest of the system. This ensures that any breaking change from the provider is caught early.

#### Cons
umer Service Code (Example)
PYTHON
import requests

class UserService:
    def __init__(self, base_url):
        self.base_url = base_url

    def get_user(self, user_id):
        response = requests.get(f"{self.base_url}/users/{user_id}")

        # Verify the contract
        if response.status_code == 200:
            data = response.json()
            assert "username" in data and isinstance(data["username"], str)
            assert "email" in data and isinstance(data["email"], str)
            return data
        else:
            raise Exception(f"Failed to get user: {response.text}")
3. Integration with CI/CD Pipeline

The verification step should be integrated into the provider's CI/CD pipeline, ensuring that any changes are validated before deployment.

#### Example
CI/CD Step
YAML
# Jenkinsfile snippet for contract testing
pipeline {
    agent any
    stages {
        stage('Contract Testing') {
            steps {
                script {
                    // Run contract verification scripts
                    sh 'python -m unittest tests.contract_tests'
                }
            }
        }
        stage('Deploy') {
            when {
                expression { return params['contract_test_passed'] } // Ensure contract test passed before deploying
            }
            steps {
                sh 'docker-compose up -d'  # Or whatever deployment command is used
            }
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

By defining the contracts clearly, implementing a robust verification mechanism in the consumer service, and integrating this into the CI/CD pipeline, we ensure that provider changes are validated before they impact consumers. This approach prevents breaking changes from propagating and ensures service reliability.

#### Common Mistakes

Failing to document the contract thoroughly.
Not running verification tests as part of the CI/CD process.
Deploying changes without adequate testing.

#### Follow-Ups

Discussing how to handle evolving contracts (e.g., adding new fields).

#### Cons
idering the impact on performance and network latency.

**Code:**

```python
class UserService:
    def __init__(self, base_url):
        self.base_url = base_url

    def get_user(self, user_id):
        response = requests.get(f"{self.base_url}/users/{user_id}")

        # Verify the contract
        if response.status_code == 200:
            data = response.json()
            assert "username" in data and isinstance(data["username"], str)
            assert "email" in data and isinstance(data["email"], str)
            return data
        else:
            raise Exception(f"Failed to get user: {response.text}")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `get_user(123)` | {"username": "John Doe", "email": "john.doe@example.com"} |

**Data:**

```yaml
openapi: 3.0.1
info:
  title: User Service
  version: "1.0"
paths:
  /users/{id}:
    get:
      summary: Retrieve a user by ID.
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User data returned.
          content:
            application/json:
              schema:
                type: object
                properties:
                  username: {type: string}
                  email: {type: string}
```

