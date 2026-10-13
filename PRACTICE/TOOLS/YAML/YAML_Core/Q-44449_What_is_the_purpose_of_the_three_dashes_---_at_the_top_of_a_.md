---
type: "practice"
difficulty: "Basic"
---

# What is the purpose of the three dashes '---' at the top of a YAML file?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Quoting and Scalars](<../../../../TOOLS/YAML/YAML_Core/Quoting_and_Scalars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The three dashes ('---') in a YAML file are used to signify the beginning of a new document, especially when working with multiple documents within a single YAML file. This is particularly useful in scenarios where you need to parse and handle more than one configuration or data block separately. For instance, if you're managing configurations for different environments like development, testing, and production, using '---' allows each environment's settings to be treated as an independent document.

In the context of YAML syntax, these dashes indicate that a new mapping (a key-value pair) is starting, but their primary function here is demarcating boundaries between documents. Let’s break down this concept further:

#### Triage
What the Question is Really Asking

The question is probing your understanding of how YAML handles multiple data blocks within a single file. You need to know that '---' acts as a delimiter, creating distinct segments.

#### Assumptions
and Reasoning from First Principles
Single Document vs Multiple Documents: A YAML file can contain one or more documents.
Delimiter Functionality: The use of '---' is a key feature in YAML for handling multiple documents.

#### Example Scenario
Consider a configuration file that needs to define settings for different environments (dev, test, prod). Using '---', you can separate these configurations.
Full Solution

When you see the following structure in a YAML file:

YAML
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest

---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
First Document: The first '---' marks the start of a new document, defining a Pod. This is akin to setting up an environment for running a pod.
Second Document: The second '---' indicates that another document follows, this time defining a Service. Here, you’re configuring how this service should be exposed and accessed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Using '---' in your YAML files helps maintain clarity and separability when dealing with complex configurations. Not using it where necessary can lead to misinterpretation of the configuration by tools that parse YAML. For fresher DevOps engineers, understanding this concept is crucial for effective use of YAML in various scenarios like deployment, configuration management, and more.

#### Common mistakes
include:

Ignoring '---' when you actually need to separate documents.
Using multiple '---' unnecessarily, which can complicate the file structure without adding any meaningful separation.

#### Example

For a more concrete example:

YAML
---
serviceA:
  replicas: 3
  image: app:v1

---
serviceB:
  replicas: 5
  image: app:v2

Here, '---' is used to separate configurations for serviceA and serviceB.

Confirmation

Correctness + completeness confirmed.

**Code:**

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest

---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
```

