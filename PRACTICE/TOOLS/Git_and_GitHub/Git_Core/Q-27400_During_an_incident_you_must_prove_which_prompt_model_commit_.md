---
type: "practice"
difficulty: "Deep"
---

# During an incident you must prove which prompt+model commit produced a harmful answer running across 200 pods. Compare baking the SHA into the image, exposing it on a probe endpoint, and reading it from logs.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reproducible Builds from Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reproducible_Builds_from_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to determine which Git commit (and hence which model version) is responsible for generating harmful responses in an AI system running across 200 pods. We'll compare three methods: baking the SHA into the image, exposing it on a probe endpoint, and reading it from logs.

#### Triage

The core of this question revolves around identifying the Git commit that generated a specific response. The interviewer is probing your understanding of deployment strategies, traceability, and logging mechanisms in an AI system. We'll need to consider how each method affects the ease of tracing, performance overhead, and potential trade-offs.

#### Solution
1. Baking the SHA into the Image

#### Assumptions

The AI model is part of a Docker image.
The image versioning process includes tagging with Git commit SHAs.

Steps:

Tagging the Image: Ensure that each Docker image tag reflects its corresponding Git commit SHA (e.g., v1.2.3@SHA).
Deployment Process: During deployment, verify that only images tagged with the correct SHAs are used.
Validation: Implement a validation step in your CI/CD pipeline to check if the deployed image's tag matches the expected commit SHA.

#### Pros

Direct and deterministic link between the codebase and the deployed model.
Easy to enforce via policies in Docker registries or Kubernetes.

#### Cons

Additional complexity in managing and updating images with new SHAs.
Potential delay in deployment if there are issues with image tags.
2. Exposing the SHA on a Probe Endpoint

#### Assumptions

The AI model runs within a containerized environment (e.g., Kubernetes).
The probe endpoint is accessible by monitoring or logging tools.

Steps:

Probe Endpoint: Implement an endpoint in your AI service that returns metadata, including the Git commit SHA.
Monitoring and Logging: Use Kubernetes probes to check the health of the pod and retrieve this metadata during incidents.
Incident Response: During a harmful response incident, query the probe endpoint to determine the exact commit SHA.

#### Pros

Lightweight solution that doesn’t affect the core application logic.
Easy integration with existing monitoring and logging practices.

#### Cons

Relies on the availability of the probe endpoint during incidents.
Potential security concerns if the endpoint is misconfigured or compromised.
3. Reading the SHA from Logs

#### Assumptions

The AI service logs include metadata about the deployed version, including Git commit SHAs.
The logging infrastructure supports querying and filtering based on specific metadata.

Steps:

Logging Configuration: Ensure that your logging setup captures relevant metadata in the logs.
Log Analysis: During an incident, analyze the logs to identify entries from pods running with the harmful response.
Metadata Extraction: Extract the Git commit SHA associated with those log entries.

#### Pros

Minimal additional overhead since it leverages existing logging mechanisms.
No need for special endpoints or image tagging.

#### Cons

Potential delay in identifying the exact commit due to the volume of logs and necessary parsing.
Risk of missing or corrupted logs, especially in a high-volume environment like 200 pods.

#### Closing Summary

Each method has its trade-offs, but baking the SHA into the image is likely the most robust approach for ensuring traceability and enforcing correct deployment. Exposing the SHA on a probe endpoint offers a lightweight solution with good monitoring support. Reading the SHA from logs is the simplest to implement but may not be as reliable in high-traffic scenarios.

#### Common Mistakes/Follow-Ups
Misunderstanding the Deployment Process: Failing to properly manage image tags or probe endpoints can lead to inconsistencies.
Overlooking Security Concerns: Not securing probe endpoints can result in unauthorized access, potentially leading to incorrect attributions.
Ignoring Log Volume: Not considering the performance impact of querying large volumes of logs during an incident response.

**Comparison:**

```
Method	Pros	Cons
Baking the SHA into the Image	Direct and deterministic link; easy to enforce policies	Additional complexity in image management
Exposing the SHA on a Probe Endpoint	Lightweight; good monitoring support	Relies on endpoint availability during incidents
Reading the SHA from Logs	Minimal overhead; leverages existing logging mechanisms	Potential delay due to log volume, risk of missing logs
```

**Code:**

```bash
# Example Dockerfile snippet with SHA baking
FROM base_image
ARG GIT_COMMIT_SHA
LABEL git.commit_sha=$GIT_COMMIT_SHA

# Example deployment script with image tagging
docker build -t myai:v1.2.3@SHA .
docker tag myai:v1.2.3@SHA myregistry/myai:v1.2.3@SHA
```

**Example input / output:**

```text
# Example log entry with metadata
2023-10-01T14:30:00Z [INFO] [AI-Service-Pod-9876543210] [Git Commit SHA: v1.2.3@SHA, Model Version: 1.2.3]
```

**Data:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ai-service-pod-1234567890
spec:
  containers:
    - name: ai-container
      image: myregistry/myai:v1.2.3@SHA
      command: ["python", "serve.py"]
---
apiVersion: monitoring.coreos.com/v1
kind: Probe
metadata:
  name: health-check
spec:
  initialDelaySeconds: 5
  periodSeconds: 10
  httpGet:
    path: /meta/commit-sha
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify Incident
6. Is SHA baked into image?
7. Retrieve exact commit
8. Check probe endpoint formetadata
9. Does log volume allow quickretrieval?
10. Read logs to find matchingcommit
11. Consider other methods ormanual inspection

