---
type: "practice"
difficulty: "Intermediate"
---

# How would you build a CI check that validates a model card / metadata (intended use, metrics, data lineage) is present and complete before promotion?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data and Analytics CI Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Data_and_Analytics_CI_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure robust model deployment pipelines, it's crucial to have a continuous integration (CI) check that verifies the presence and completeness of a Model Card or metadata before promoting a model. This ensures that all necessary information is available, supporting transparent and ethical use of the model.

#### Triaging the Question

The question asks for building a CI check that validates specific metadata fields in a Model Card—such as intended use, metrics, and data lineage—are present and complete. The key idea here is to create a script or tool that can automatically inspect these metadata files and flag any issues.

#### Assumptions
and Context
Model Card Format: Assume the Model Card is stored in a structured format such as JSON.
Metadata Fields: These fields include intended use, performance metrics (e.g., accuracy, F1 score), and data lineage details.
CI/CD Pipeline: We are working within an existing CI/CD pipeline that stages models for deployment.
Step-by-Step Solution
1. Define the Model Card Schema

First, define the schema of the expected Model Card to ensure all required fields are present:

JSON
{
  "intendedUse": string,
  "performanceMetrics": {
    "accuracy": number,
    "f1Score": number,
    ...
  },
  "dataLineage": [
    {
      "sourceData": string,
      "processingSteps": [string],
      ...
    }
  ],
  ...
}
2. Create a Validation Script

Next, develop a script that reads the Model Card and checks for completeness:

PYTHON
import json

def validate_model_card(model_card_path):
    with open(model_card_path, 'r') as f:
        model_card = json.load(f)

    required_fields = ["intendedUse", "performanceMetrics", "dataLineage"]

    # Check if all fields are present
    for field in required_fields:
        if field not in model_card:
            return False, f"Missing {field} field"

    # Validate performance metrics structure
    if 'accuracy' not in model_card['performanceMetrics'] or 'f1Score' not in model_card['performanceMetrics']:
        return False, "Performance metrics are incomplete"

    # Validate data lineage presence and format
    for entry in model_card.get("dataLineage", []):
        if 'sourceData' not in entry or 'processingSteps' not in entry:
            return False, "Data lineage entries are missing required fields"

    return True, None

# Example usage
model_card_path = "path/to/model_card.json"
is_valid, error_msg = validate_model_card(model_card_path)
if is_valid:
    print("Model card is valid")
else:
    print(f"Model card validation failed: {error_msg}")
3. Integrate the Script into CI/CD

Finally, integrate this script into your CI/CD pipeline as part of the model promotion stage. This can be done using Jenkins Pipeline, GitLab CI, or any other CI tool.

GROOVY
pipeline {
    agent any

    stages {
        stage('Model Validation') {
            steps {
                script {
                    def isValid = bat(returnStatus: true, script: "python3 validate_model_card.py")
                    if (isValid == 1) {
                        error "Validation failed"
                    }
                }
            }
        }
        // Other stages like testing and deployment
    }
}

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures that all necessary metadata is present before a model can be promoted, enhancing the transparency and accountability of the deployed models. Common mistakes include not defining clear validation rules or neglecting to automate checks in the CI/CD pipeline.

For follow-ups:

Enhancing Validation Rules: Consider adding more specific validations like checking metric ranges.
Error Handling: Improve error handling by providing detailed logs for easier troubleshooting.
Security Considerations: Ensure that sensitive data is not included in the Model Card and that access to these files is restricted appropriately.

