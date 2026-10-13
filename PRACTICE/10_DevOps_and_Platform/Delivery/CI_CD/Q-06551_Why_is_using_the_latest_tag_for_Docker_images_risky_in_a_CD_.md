---
type: "practice"
difficulty: "Intermediate"
---

# Why is using the 'latest' tag for Docker images risky in a CD pipeline?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Container Images and Registries](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Container_Images_and_Registries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Using latest as the Docker image tag can introduce significant risks in a Continuous Delivery (CD) pipeline due to its dynamic nature and potential for unexpected updates. The key idea is that using latest means your application will always get the latest version of the image, which might not be what you expect or want.

Firstly, let's triage the question: The interviewer is probing about the risks associated with using the latest tag in a CI/CD pipeline and asking for a detailed explanation. We need to discuss how this can lead to unexpected behavior, potential downtime, and increased complexity.

#### Assumptions

You have a Docker-based application deployed via a CI/CD pipeline.
The application is critical, and changes in the latest image could affect its operation.

Now let's reason step by step from first principles:

Dynamic Nature of latest: When you use latest, it always pulls the most recent version of the Docker image. This means that any changes made to the Dockerfile, including breaking changes or unintended features, will be immediately deployed. For example, if a developer adds a new dependency or makes a change that impacts the application's behavior, using latest ensures this change is pushed to production.

Risk of Breaking Changes: Using latest can lead to unexpected and potentially harmful updates. A critical bug fix in the image could inadvertently break your application, leading to downtime or even service outages. Conversely, a developer might push an experimental feature that hasn't been thoroughly tested yet, which could cause issues.

Versioning Best Practices: To mitigate these risks, it is recommended to use semantic versioning (e.g., v1.2.3) for Docker images. This approach allows you to control the deployment of specific versions and roll back if necessary. Using tags like v1.0, v1.1, etc., ensures that you can pinpoint exactly which version your application is using, reducing the risk of unexpected changes.

Rollout Strategy: Instead of deploying directly from latest, you should use a rolling update strategy in your CI/CD pipeline. This involves gradually deploying new versions to ensure stability and allow for easy rollbacks if something goes wrong. For instance, you could deploy 10% of the instances with the new version and monitor them before rolling out the rest.

Tag Management: Effective tag management is crucial. You should have a clear process for updating tags and ensuring that only tested versions are pushed to production. Automation tools can help in managing this process efficiently, such as tagging images after successful tests or deployments.

Testing Environments: Proper testing environments must be set up to validate changes before they reach the production environment. This includes integration testing, unit testing, and possibly performance testing to ensure that new versions behave as expected.

In summary, using latest in a CI/CD pipeline can lead to unexpected behavior and increased risk due to its dynamic nature. To mitigate these risks, it is better to use versioned tags, implement a robust rollout strategy, manage tags effectively, and set up thorough testing environments.

**Comparison:**

```
Tag Usage	Risk	Mitigation
latest	High	Use semantic versioning, rollouts, and testing environments.
v1.2.3	Low	Rollouts, testing, and clear processes for tag management.
```

**Code:**

```
Example of semantic versioning in Dockerfile tags

docker build -t myapp:v1.2.3 .

CODE
Copy

```bash
# Example of using a tag in deployment script
kubectl apply -f deployment.yaml --image=myapp:v1.2.3
```

**Flow:**

1. CI Pipeline
2. Testing
3. Rollout Strategy
4. Production

