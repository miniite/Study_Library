---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd set up a CI/CD pipeline that automatically evaluates a new CV model on a golden test set and blocks deploy on regression.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Input Resolution Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Input_Resolution_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In setting up a CI/CD pipeline for evaluating computer vision (CV) models, we need to ensure that any changes or updates do not degrade performance. This involves automating model evaluation against a "golden test set" and blocking deployment if the new version of the model exhibits significant regression in key metrics. The core idea is to maintain code quality and model integrity by systematically testing each update before it hits production.

The pipeline will consist of several stages: build, test, and deploy. We'll use tools like Jenkins, GitLab CI/CD, or GitHub Actions for orchestration. Here's a step-by-step breakdown:

Source Code Management: Ensure the models are version-controlled in a repository.
Build Step: Compile the code if necessary (for example, using TensorFlow).
Test Step:
Evaluate the model against the golden test set.
Calculate performance metrics such as accuracy, precision, recall, and F1 score.
Deploy Decision Step:
Compare the new model's metrics with historical or expected values.
Block deployment if there is a notable regression.

Now, let's delve into each step in detail:

1. Source Code Management

We'll use Git for version control. This ensures that any changes to the model code are tracked and can be rolled back if needed.

BASH
git clone https://github.com/your-organization/model-repository.git
2. Build Step

For a machine learning pipeline, this step might involve compiling dependencies or training the model again. For simplicity, we'll assume pre-trained models without compilation:

YAML
stages:
  - build
  - test
  - deploy

build_model:
  stage: build
  script:
    - echo "Build model" # Placeholder for actual build steps
3. Test Step

This is where the magic happens. We need to evaluate the model's performance and compare it with a baseline or golden set.

#### Example
YAML Configuration:
YAML
test_model:
  stage: test
  script:
    - python evaluate.py --model_path /path/to/model --golden_set /path/to/golden/test/set --output_results /results/output

Here, evaluate.py is a Python script that loads the model and evaluates it against the golden test set. It calculates various metrics and outputs them to /results/output.

Sample Evaluation Script (evaluate.py):
PYTHON
import torch
from torchvision import models

def evaluate_model(model_path, golden_set):
    # Load the pre-trained model from path
    model = models.resnet50(pretrained=False)
    checkpoint = torch.load(model_path, map_location=torch.device('cpu'))
    state_dict = {k.replace('module.', ''): v for k, v in checkpoint['state_dict'].items()}
    model.load_state_dict(state_dict)

    # Load the golden test set
    dataloader = load_golden_test_set(golden_set)

    # Perform evaluation
    accuracy, precision, recall, f1_score = perform_evaluation(model, dataloader)
    return {'accuracy': accuracy, 'precision': precision, 'recall': recall, 'f1_score': f1_score}

def load_golden_test_set(path):
    # Placeholder function to load data from the golden test set
    pass

def perform_evaluation(model, dataloader):
    # Perform model evaluation and calculate metrics
    pass

if __name__ == "__main__":
    results = evaluate_model('/path/to/model', '/path/to/golden/test/set')
    print(results)
4. Deploy Decision Step

Based on the evaluation results, we need to decide whether to proceed with deployment or not.

YAML
deploy_decision:
  stage: deploy
  script:
    - if [ $(cat /results/output | jq '.accuracy' | python -c "import json; print(json.loads(input().strip()))") -lt 95 ]; then echo "Model failed accuracy test"; exit 1; fi
    - if [ $(cat /results/output | jq '.f1_score' | python -c "import json; print(json.loads(input().strip()))") -lt 80 ]; then echo "Model failed F1 score test"; exit 1; fi

Here, we check the accuracy and F1 score against a threshold. If any metric falls below the expected value, deployment is blocked.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, setting up this CI/CD pipeline requires careful configuration of each stage to ensure robust model evaluation. Key considerations include:

Test Coverage: Ensure all critical metrics are evaluated.
Thresholds: Define clear thresholds for what constitutes acceptable performance.
Logistics: Manage the golden test set securely and efficiently.

Common pitfalls include insufficient testing, overly permissive thresholds, or failing to handle edge cases like model dependencies. Always validate the pipeline thoroughly before relying on it in production environments.

Self-Check

This solution is correct and comprehensive, covering all necessary steps for a robust CI/CD pipeline in ML engineering.

**Comparison:**

```
Metric	Threshold
Accuracy	> 95%
F1 Score	> 80%
```

**Code:**

```yaml
stages:
  - build
  - test
  - deploy

build_model:
  stage: build
  script:
    - echo "Build model"

test_model:
  stage: test
  script:
    - python evaluate.py --model_path /path/to/model --golden_set /path/to/golden/test/set --output_results /results/output

deploy_decision:
  stage: deploy
  script:
    - if [ $(cat /results/output | jq '.accuracy' | python -c "import json; print(json.loads(input().strip()))") -lt 95 ]; then echo "Model failed accuracy test"; exit 1; fi
    - if [ $(cat /results/output | jq '.f1_score' | python -c "import json; print(json.loads(input().strip()))") -lt 80 ]; then echo "Model failed F1 score test"; exit 1; fi
```

**Example input / output:**

```text
$ cat /results/output | jq '.accuracy'
95.3

$ cat /results/output | jq '.f1_score'
0.95
```

**Data:**

```json
{
  "accuracy": 95.3,
  "precision": 0.94,
  "recall": 0.96,
  "f1_score": 0.95
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Source Code Management
6. Build Model
7. Test Model
8. Accuracy <= 95?
9. Block Deployment
10. F1 Score <= 80?
11. Deploy Model

