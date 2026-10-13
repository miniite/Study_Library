---
type: "practice"
difficulty: "Intermediate"
---

# How do you nest containers under a Pod template inside a Deployment YAML? Describe the path of keys.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multi-document YAML for Kubernetes](<../../../../TOOLS/YAML/YAML_Core/Multi-document_YAML_for_Kubernetes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Kubernetes, deploying applications often involves using Deployment resources to manage and scale your application pods. Sometimes, it's necessary to nest containers within a single pod template for various reasons, such as sharing resources or needing multiple containers that work closely together.

TL;DR:

To nest containers under a Pod template inside a Deployment YAML, you need to define the container specifications within the spec.template.spec.containers section. Each container has its own configuration like image, command, and ports. The key path for this setup in your Deployment YAML is:

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: container1
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: container2
        image: php-fpm:latest
        ports:
        - containerPort: 9000

#### Triage

The question is asking how to structure a Deployment YAML file in Kubernetes, specifically focusing on how to place multiple containers within the same pod template. The interviewer might be probing your understanding of Kubernetes resource definitions and nested structures.

Key Assumptions:
You are familiar with basic Kubernetes resources like Deployments, Pods, and Containers.
Understanding of YAML syntax and structure is required.
Knowledge about containerization, networking, and shared resources in a pod context.
Step-by-Step Solution From First Principles:

Define the Deployment Resource:

Start by defining the top-level resource, which is a Deployment. This involves specifying the API version, kind of object (in this case, Deployment), and giving it a name.

Replicas Specification:

Specify how many replicas should be created for the application using the replicas field in the spec.

Template Definition:

Use the template field to define the pod template that is going to run.
Inside the template, add metadata like labels, which help you identify the pods.

Pod Spec and Containers:

Define the spec of the pod template under metadata.
Within this spec, list out multiple containers using the containers array.
Each container requires fields like name, image, and ports.
Sample Deployment YAML

Here’s a sample configuration for nesting two containers inside a single Pod:

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: container1
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: container2
        image: php-fpm:latest
        ports:
        - containerPort: 9000

#### Closing Summary
and Common Mistakes/Follow-Ups:

This YAML defines a Deployment that creates three replicas of pods. Each pod contains two containers: one running Nginx (a web server) and the other running PHP-FPM (PHP FastCGI Process Manager). Both containers are configured to expose their respective ports.

#### Common Mistakes

Forgetting to define the containers array within the pod spec.
Not specifying the correct field names like image, name, or ports.
Missing the necessary metadata for the pod template.

#### Follow-Ups

How would you handle resource sharing between these containers?
What if one container depends on another? How do you manage this in a Pod?

**Code:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: container1
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: container2
        image: php-fpm:latest
        ports:
        - containerPort: 9000
```

