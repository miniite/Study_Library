---
type: "practice"
difficulty: "Basic"
---

# How do you write a boolean value in a Kubernetes YAML such as disabling a probe or setting privileged to false?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multi-document YAML for Kubernetes](<../../../../TOOLS/YAML/YAML_Core/Multi-document_YAML_for_Kubernetes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Kubernetes, YAML is used to define objects like Deployments, Pods, and ConfigMaps. When it comes to writing a boolean value in a YAML file, it's quite straightforward but requires attention to detail to ensure that the YAML syntax is correct.

The question asks specifically about setting up booleans within a Kubernetes YAML configuration. For example, you might want to disable a probe (like liveness or readiness) or set privileged to false for security reasons. Let's break this down step-by-step:

Understanding Booleans in YAML: In YAML, boolean values are represented as true and false. These are case-sensitive.
Disabling a Probe:
You might want to disable a probe in a Pod or Deployment configuration. This is often used when you don’t want the Kubernetes controller to check if your application is alive.
Setting Privileged Mode: The privileged field in Pod specifications controls whether a container can use host privileges, such as access to raw sockets.

Let's see an example of how these might be implemented:

YAML
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo 'Hello from Kubernetes'; sleep 5; done"]
    ports:
    - containerPort: 8080
  # Disabling liveness and readiness probes
  livenessProbe:
    exec:
      command:
        - /bin/bash
        - -c
        - exit 1
    initialDelaySeconds: 30
    periodSeconds: 10
    failureThreshold: 5

  privileged: false # Setting the container to non-privileged mode

In this example, we see that livenessProbe and privileged are set as boolean values. The livenessProbe is disabled by setting an invalid command (exit 1) with a high failure threshold.

**Code:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo 'Hello from Kubernetes'; sleep 5; done"]
    ports:
    - containerPort: 8080
  # Disabling liveness and readiness probes
  livenessProbe:
    exec:
      command:
        - /bin/bash
        - -c
        - exit 1
    initialDelaySeconds: 30
    periodSeconds: 10
    failureThreshold: 5

  privileged: false # Setting the container to non-privileged mode
```

