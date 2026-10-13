---
type: "practice"
difficulty: "Expert"
---

# A deploy shipped a model that's better on average but much worse on your top enterprise customer's queries; how would you build customer-specific eval gates into CD?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Performance Regression Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Performance_Regression_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an AI deployment scenario, it’s critical to ensure that models perform well across all customers while also addressing individual needs. When a new model performs better on average but much worse on a specific customer's queries, we need robust customer-specific evaluation mechanisms as part of our CI/CD pipeline. This involves integrating custom evaluation gates directly into the continuous deployment process.

#### Triaging the Question

The core of this question is about implementing a dynamic evaluation mechanism that can adapt to different customer requirements without impacting overall deployment speed or quality. The interviewer likely wants you to explore how to design, implement, and monitor such a system. Key assumptions include:

There are multiple customers with varying needs.
Performance metrics differ significantly between customers.
The current CI/CD pipeline must be extended to support these evaluations.

#### First Principles Solution

To build customer-specific eval gates into the CI/CD process, we need to follow these steps:

Identify Customer-Specific Metrics: Define performance metrics that are relevant for each customer. These could include accuracy, latency, or specific error rates.
Extend the Build Pipeline: Integrate evaluation stages in the build pipeline to test models against each customer's criteria.
Implement Custom Evaluation Gates: Use conditional logic within the CI/CD flow to trigger additional evaluations based on the model’s performance across different metrics.
Detailed Solution

Let's break down each step:

Define Customer-Specific Metrics:

For a top enterprise customer, we might define metrics such as 99th percentile latency, 95% accuracy, and error rate on critical queries.
These can be captured from historical data or defined by the customer’s service level agreements (SLAs).

Extend the Build Pipeline:

Integrate a stage in the CI/CD pipeline where models are tested against these metrics.
This could involve running a subset of test cases that simulate the enterprise customer's queries.

Implement Custom Evaluation Gates:

Use a decision-making framework to evaluate whether the model meets the specific criteria for each customer.
If it doesn't, fail the deployment and allow for manual review or rollback.

#### Example
Implementation

Here’s an example of how this could be implemented using Jenkins and Python scripts:

PYTHON
# Define evaluation functions
def check_latency(model_outputs):
    # Check 99th percentile latency
    if max(model_outputs) > MAX_LATENCY_THRESHOLD:
        return False

def check_accuracy(model_outputs, true_labels):
    accuracy = calculate_accuracy(model_outputs, true_labels)
    if accuracy < MIN_ACCURACY_THRESHOLD:
        return False

# Define the evaluation gate in Jenkins
pipeline {
    stages {
        stage('Model Build') {
            steps {
                script {
                    // Train and save the model
                    def model_outputs = train_model()
                    def true_labels = get_true_labels()

                    // Evaluate against customer-specific metrics
                    if (check_latency(model_outputs) and check_accuracy(model_outputs, true_labels)) {
                        echo 'Evaluation passed'
                        archiveArtifacts artifacts: 'model.pkl', allowEmptyArchive: true
                    } else {
                        error 'Evaluation failed'
                    }
                }
            }
        }
    }
}

#### Closing Summary
and Common Mistakes

This approach ensures that the model is thoroughly evaluated against specific customer requirements before deployment. It also allows for flexibility in tailoring the evaluation to different customers.

#### Common Mistakes

Failing to define clear, measurable metrics.
Overlooking edge cases where models might perform well on average but poorly under certain conditions.
Not implementing a robust rollback mechanism if eval gates fail.

By addressing these points and ensuring comprehensive testing, we can significantly reduce the risk of deploying suboptimal models to critical customers.

**Code:**

```
GROOVY
Copy
pipeline {
    stages {
        stage('Model Build') {
            steps {
                script {
                    // Train and save the model
                    def model_outputs = train_model()
                    def true_labels = get_true_labels()

                    // Evaluate against customer-specific metrics
                    if (check_latency(model_outputs) && check_accuracy(model_outputs, true_labels)) {
                        echo 'Evaluation passed'
                        archiveArtifacts artifacts: 'model.pkl', allowEmptyArchive: true
                    } else {
                        error 'Evaluation failed'
                    }
                }
            }
        }
    }
}
```

**Example input / output:**

```text
Input Model Outputs and True Labels:
model_outputs = [[0.1, 0.9], [0.8, 0.2]]
true_labels = [[1, 0], [0, 1]]

Output:
Evaluation passed
```

**Data:**

```yaml
# Example of model outputs and true labels
model_outputs:
  - [0.1, 0.9]
  - [0.8, 0.2]

true_labels:
  - [1, 0]
  - [0, 1]
```

