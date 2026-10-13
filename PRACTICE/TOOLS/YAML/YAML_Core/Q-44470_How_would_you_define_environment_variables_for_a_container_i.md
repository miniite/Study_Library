---
type: "practice"
difficulty: "Intermediate"
---

# How would you define environment variables for a container inside a Kubernetes Pod YAML spec?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multi-document YAML for Kubernetes](<../../../../TOOLS/YAML/YAML_Core/Multi-document_YAML_for_Kubernetes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Environment variables are crucial in managing configurations that can be set at runtime and applied across multiple instances of an application within a Kubernetes cluster. When defining these environment variables in a Kubernetes Pod's YAML specification, we need to ensure they can be accessed by the containers running inside the pod. This involves understanding how to specify them in the spec section of the Pod YAML file.

To start, let’s define what environment variables are and why they are important:

Environment variables allow us to configure applications dynamically without changing their code.
They can store sensitive information like API keys or database credentials securely.
They enable easier deployment across different environments (development, testing, production).

#### Triage
Restating the Question

The question is asking how to properly define environment variables for containers running inside a Kubernetes Pod. The key aspects include:

Identifying where in the YAML file these variables are specified.
Ensuring the values can be securely stored and passed to the containers.
Understanding any best practices or conventions related to this.

#### First Principles Solution

The core of setting up environment variables in Kubernetes involves using the env field within the container specification. Here’s a step-by-step approach:

Define Environment Variables:
Use the env field inside each container definition.
Securely Store Values:
For sensitive information, use secrets and config maps to store these values securely.
Accessing Variables:
Ensure that the containers have access to these variables via their environment.

#### Example
YAML Specification

Here is an example of how you might define environment variables in a Kubernetes Pod YAML file:

YAML
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    env:
      - name: MY_API_KEY
        value: "my_api_key_value"
    # For sensitive information, use secrets or config maps
    # - name: SECRET_ENV_VAR
    #   valueFrom:
    #     secretKeyRef:
    #       name: my-secret
    #       key: api-key

  restartPolicy: OnFailure

#### Closing Summary
and Common Mistakes/Follow-Ups
Ensure that sensitive information is stored securely, either in secrets or config maps.
Use valueFrom for complex variable values like files or environment variables from other sources.

#### Cons
ider using labels and annotations to manage environment-specific configurations.

#### Common mistakes
include:

Storing sensitive information directly in the YAML file without encryption.
Not using secrets or config maps for sensitive data, which can lead to security risks.
Failing to restart the container if an environment variable changes, leading to stale configuration.

**Code:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-image
    env:
      - name: MY_API_KEY
        value: "my_api_key_value"
```

