---
type: "practice"
difficulty: "Deep"
---

# Why did you choose an ML-specific pipeline tool (Kubeflow/Vertex/SageMaker Pipelines) over plain GitHub Actions for a model workflow, in your experience?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ ML Model Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In my experience, choosing an ML-specific pipeline tool like Kubeflow, Vertex AI Pipelines, or SageMaker Pipelines over plain GitHub Actions for a model workflow offers several advantages that are crucial for maintaining robust, scalable, and maintainable machine learning (ML) projects. These tools are designed specifically with the complexity of ML workflows in mind, providing features and best practices tailored to this domain.

Firstly, let's break down what these tools are really asking: The question is probing into the specific needs of an ML workflow that plain GitHub Actions might not fully address. We need to consider aspects like resource management, model deployment, scalability, monitoring, and collaboration within a team. I'll start by detailing why plain GitHub Actions fall short in handling these requirements.

**Code:**

```
Example of a basic GitHub Actions workflow for an ML project

name: Train Model on: push: branches: - main

jobs: train_model: runs-on: ubuntu-latest steps: - name: Checkout repository uses: actions/checkout@v3 - name: Setup Python environment run: | python -m pip install --upgrade pip pip install -r requirements.txt - name: Train model run: python train.py
```

