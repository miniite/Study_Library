---
type: "practice"
difficulty: "Deep"
---

# A domain insists its feature product is 'done' after one publish, but ML feature products need continuous operation. How do you architect and fund the ongoing operational model (on-call, freshness SLAs, deprecation) so feature products don't silently rot?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product SLAs](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Data_Product_SLAs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of ensuring that an ML feature product remains continuously operational, we need to design a robust data architecture and establish clear funding and operational models. The key is to create a resilient system where feature products are regularly maintained, with defined SLAs for freshness and deprecation. Here’s how we can achieve this.

Firstly, it's crucial to understand that ML features are living assets that require continuous care and attention to remain relevant and effective. They should not be treated as one-time deliverables but rather as ongoing projects that need regular updates and maintenance. This means setting up a structured operational model for on-call support, maintaining data freshness, and ensuring proper deprecation processes.

Secondly, the architecture needs to be designed with a "data mesh" approach in mind, where data is shared across teams while still providing governance and control. This will enable better collaboration and faster feedback cycles, which are essential for continuous improvement of ML features.

Now, let’s break down the solution step by step:

Define Clear SLAs: Establish Service Level Agreements (SLAs) that outline expected data freshness, availability, and quality. For example, we might require that data is refreshed at least once a day or that models are retrained every week.

Implement Continuous Deployment Pipelines: Set up CI/CD pipelines to automate the deployment of new features and updates. This ensures that any changes can be quickly rolled out without manual intervention, reducing the risk of downtime.

On-Call Rotations: Design a on-call rotation system where key stakeholders are responsible for monitoring the system and responding to issues. This should include both technical experts and business representatives to ensure a holistic view.

Data Versioning and Deprecation Policies: Implement versioning mechanisms to track changes in data models and features. Define deprecation policies that outline when and how old versions of features will be phased out. For example, if a new feature is released, the previous one might be deprecated after 6 months.

Budgeting for Continuous Operations: Secure dedicated funding for ongoing operations. This could come from a mix of internal budgets, cost-center allocations, or even external funding sources like partnerships or grants focused on continuous innovation in ML.

By following these steps, we can create a sustainable and efficient operational model that ensures feature products remain continuously relevant and effective.

**Code:**

```python
# Example CI/CD Pipeline Integration with Jenkins

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def modelVersion = "v1"
                    echo "Building model version ${modelVersion}"
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    echo "Running tests for model v1"
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying model version ${modelVersion} to production"
                }
            }
        }
    }
}
```

**Example input / output:**

```text
Input:
- A request to monitor and update a feature product with version v1.0

Output:
- Monitoring logs show that the data was last refreshed on 2023-09-15.
- The model is currently at version v1.0, which will be retired on 2023-10-01 and replaced by v2.0.

Dry-run:
- Version v1.0 of the feature product is still in use but will reach its retirement date soon.
- A new version v2.0 is being developed and tested to replace it.
```

**Data:**

```json
{
  "onCall": [
    {
      "name": "John Doe",
      "team": "Data Science",
      "shift": "Day Shift"
    },
    {
      "name": "Jane Smith",
      "team": "Engineering",
      "shift": "Night Shift"
    }
  ],
  "freshnessSLA": {
    "dataRefreshRate": "Daily",
    "modelRetrainingFrequency": "Weekly"
  },
  "deprecationPolicy": {
    "version": "v1.0",
    "retirementDate": "2023-10-01",
    "replacedByVersion": "v2.0"
  }
}
```

