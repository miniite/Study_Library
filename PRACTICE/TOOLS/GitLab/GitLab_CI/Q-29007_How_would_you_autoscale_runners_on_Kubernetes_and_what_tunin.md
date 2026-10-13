---
type: "practice"
difficulty: "Expert"
---

# How would you autoscale runners on Kubernetes and what tuning controls concurrency and cost?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ GitLab CI Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/GitLab_CI_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To autoscale GitLab CI runners on Kubernetes, we need to leverage Kubernetes' powerful auto-scaling capabilities and carefully configure our runners to ensure efficient use of resources while keeping costs in check. The key steps involve setting up a GitLab CI runner service that runs as a Kubernetes pod, configuring the necessary scaling policies, and tuning concurrency limits.

#### Triaging the Question

The question is probing how to implement autoscaling for GitLab CI runners on Kubernetes, focusing on two main aspects: (1) how to automatically scale the number of runners based on demand, and (2) how to control concurrency to balance performance and cost. The interviewer may be interested in understanding your familiarity with Kubernetes auto-scaling mechanisms, GitLab CI configurations, and best practices for managing resources.

#### First-Principles Solution
1. Set Up the Runner Service

First, we need to deploy a GitLab CI runner as a Kubernetes pod. This involves creating a Deployment manifest that specifies the runner image and any necessary environment variables or secrets required by GitLab CI.

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab-ci-runner
spec:
  replicas: 0 # Initial number of replicas, set to zero for auto-scaling
  selector:
    matchLabels:
      app: gitlab-ci-runner
  template:
    metadata:
      labels:
        app: gitlab-ci-runner
    spec:
      containers:
      - name: runner
        image: registry.gitlab.com/gitlab-org/github-actions-runner:latest # Use the appropriate runner image
        env:
          - name: GITLAB_URL
            value: "https://your-gitlab-instance.com"
          - name: CI_PROJECT_ID
            valueFrom:
              secretKeyRef:
                name: gitlab-ci-secrets
                key: ci_project_id
          - name: RUNNER_REGISTRATION_TOKEN
            valueFrom:
              secretKeyRef:
                name: gitlab-ci-secrets
                key: runner_registration_token
        resources:
          limits:
            cpu: "1"
            memory: 512Mi
          requests:
            cpu: "0.5"
            memory: 256Mi
2. Define Autoscaling Mechanisms

To autoscale the runners, we can use Horizontal Pod Autoscaler (HPA) based on metrics such as CPU utilization or custom metrics like job queue length. This involves creating an HPA that scales up and down based on the current workload.

YAML
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: gitlab-ci-runner-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: gitlab-ci-runner
  minReplicas: 0
  maxReplicas: 10 # Set a reasonable upper limit based on your infrastructure and resource constraints
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 80 # Scale up or down when the average CPU utilization exceeds 80%
3. Tuning Concurrency Limits

To control concurrency, we set limits in the runner configuration within GitLab CI. This ensures that a single runner doesn't take on too many jobs simultaneously, which can lead to resource exhaustion and poor performance.

In your .gitlab-ci.yml, you can specify job limits:

YAML
stages:
  - build

build:
  stage: build
  script:
    - echo "Building..."
  concurrency: 5 # Limit the number of concurrent builds per runner

#### Closing Summary

Autoscaling GitLab CI runners on Kubernetes involves deploying a runner service, configuring HPA to scale based on metrics, and tuning job concurrency limits. This approach ensures efficient use of resources while maintaining performance and cost efficiency.

#### Common Mistakes/Follow-Ups
Not Considering Cost Implications: Autoscaling can be expensive if not managed properly, especially with resource-intensive jobs.
Overlooking Job Dependencies: Ensuring that jobs are scheduled correctly to avoid overloading the runners or leaving them idle.
Ignoring Performance Metrics: Monitoring CPU and memory usage to adjust HPA parameters as needed.

**Code:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab-ci-runner
spec:
  replicas: 0 # Initial number of replicas, set to zero for auto-scaling
  selector:
    matchLabels:
      app: gitlab-ci-runner
  template:
    metadata:
      labels:
        app: gitlab-ci-runner
    spec:
      containers:
      - name: runner
        image: registry.gitlab.com/gitlab-org/github-actions-runner:latest # Use the appropriate runner image
        env:
          - name: GITLAB_URL
            value: "https://your-gitlab-instance.com"
          - name: CI_PROJECT_ID
            valueFrom:
              secretKeyRef:
                name: gitlab-ci-secrets
                key: ci_project_id
          - name: RUNNER_REGISTRATION_TOKEN
            valueFrom:
              secretKeyRef:
                name: gitlab-ci-secrets
                key: runner_registration_token
        resources:
          limits:
            cpu: "1"
            memory: 512Mi
          requests:
            cpu: "0.5"
            memory: 256Mi

---

apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: gitlab-ci-runner-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: gitlab-ci-runner
  minReplicas: 0
  maxReplicas: 10 # Set a reasonable upper limit based on your infrastructure and resource constraints
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 80 # Scale up or down when the average CPU utilization exceeds 80%
```

