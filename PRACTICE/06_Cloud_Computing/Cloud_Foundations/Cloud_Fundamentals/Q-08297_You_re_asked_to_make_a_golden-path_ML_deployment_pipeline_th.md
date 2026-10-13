---
type: "practice"
difficulty: "Deep"
---

# You're asked to make a golden-path ML deployment pipeline that 90% of model teams adopt by default while the 10% with genuine research needs can deviate without abandoning the platform's cost and safety guarantees. How do you architect the opt-out?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a golden-path ML deployment pipeline that 90% of model teams adopt by default while allowing the remaining 10% to deviate without abandoning cost and safety guarantees, we need to strike a balance between standardization and flexibility. The key idea is to build a robust platform with default configurations optimized for common use cases but also provide fine-grained control options where needed.

First, let's triage the question: The interviewer is probing how you can design a system that is both scalable and flexible enough to meet diverse requirements while maintaining consistency in terms of cost management and security. We need to assume that the platform will have various layers including data preprocessing, model training, hyperparameter tuning, and deployment.

The first-principles approach involves breaking down the pipeline into modular components, ensuring each step can be customized without disrupting the overall integrity of the system. Here's a step-by-step solution:

Default Pipeline Design: Start by designing a default ML deployment pipeline that covers common use cases. This includes data ingestion, preprocessing, model training, validation, and deployment.

Modular Components: Each component in the pipeline should be modular and configurable. For instance:

Data Ingestion: Support multiple sources like S3, BigQuery, etc.
Preprocessing: Allow customizable transformations but provide defaults for common tasks.
Training Pipeline: Use hyperparameter tuning and model validation steps with default configurations that can be overridden.

Pipeline Orchestration: Use a container orchestration platform (e.g., Kubernetes) to manage the execution of these components. This ensures each step is isolated, scalable, and maintainable.

Default vs. Custom Configurations: Implement a configuration management system where users can opt into default configurations or specify custom configurations. The system should enforce certain rules to ensure cost and security are maintained even when customizations are made.

Cost Management and Security Policies: Define clear policies around resource allocation, encryption, access controls, and logging. These policies should be enforced by the platform regardless of whether a team uses defaults or customizes their pipeline.

Opt-Out Mechanism: For those needing to deviate from the default path, provide a streamlined process for requesting customization. This could involve submitting a ticket or using an API endpoint that logs and approves changes before they take effect.

Monitoring and Feedback Loop: Continuously monitor the performance and security of both the default and customized pipelines. Use this data to refine the default configurations based on feedback from users and automated tests.

#### Closing Summary
By building a modular, configurable, and well-monitored platform, we can ensure that most model teams adopt the standard pipeline while allowing exceptions with clear governance mechanisms in place. This approach maintains cost efficiency and security standards across all deployments, even as teams experiment with advanced configurations to meet specific research needs.

#### Common Mistakes/Follow-Ups

Failing to define clear policies around customization requests.
Not implementing robust monitoring and feedback loops for performance and security.
Overlooking the importance of user onboarding and support in adopting the default pipeline.

**Comparison:**

```
Component	Default Configurations	Customization Options
Data Ingestion	AWS S3, BigQuery	Other sources, custom scripts
Preprocessing	Standard scaling, feature selection	User-defined transformations
Training	Hyperparameter tuning with default settings	Custom hyperparameters, advanced tuning
Evaluation	Cross-validation, accuracy metrics	Custom evaluation methods
Deployment	Auto-scaling, A/B testing	Custom deployment strategies
```

**Code:**

```python
# Example Python code snippet for a simple model deployment process
import kfp

@kfp.dsl.pipeline(name="Default ML Pipeline")
def ml_pipeline():
    data_ingestion_op = dsl.ContainerOp(
        name="Data Ingestion",
        image="your-data-ingestion-image"
    )

    preprocessing_op = dsl.ContainerOp(
        name="Preprocessing",
        image="your-preprocessing-image"
    )

    training_op = dsl.ContainerOp(
        name="Training",
        image="your-training-image"
    )

    evaluation_op = dsl.ContainerOp(
        name="Evaluation",
        image="your-evaluation-image"
    )

    serving_op = dsl.ContainerOp(
        name="Serving",
        image="your-serving-image"
    )

kfp.compiler.Compiler().compile(ml_pipeline, "default-pipeline.yaml")
```

**Example input / output:**

```text
# Example input to output for a request process
Request: Create a custom preprocessing step for the default pipeline.
Response: Custom configuration approved. New step added in the workflow with monitoring enabled.

Request: Need more resources for the training phase.
Response: Increased resource allocation for the specified deployment. Monitoring and security checks passed.
```

**Data:**

```yaml
# Example Kubernetes Deployment YAML for a Model Serving Component
apiVersion: apps/v1
kind: Deployment
metadata:
  name: model-serving-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: model-serving
  template:
    metadata:
      labels:
        app: model-serving
    spec:
      containers:
      - name: model-serving-container
        image: your-model-serving-image
        ports:
        - containerPort: 8080
```

**Flow:**

1. Data Ingestion
2. Preprocessing
3. Training
4. Evaluation
5. Serving
6. Custom Config Request
7. Pipeline ReconfigurationWorkflow
8. Maintenance and SecurityChecks
9. Approval Flow
10. Update Pipeline Configuration
11. Fallback to Default

