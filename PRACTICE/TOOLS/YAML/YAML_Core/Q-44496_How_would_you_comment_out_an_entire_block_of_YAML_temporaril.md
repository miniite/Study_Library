---
type: "practice"
difficulty: "Basic"
---

# How would you comment out an entire block of YAML temporarily during debugging?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Quoting and Scalars](<../../../../TOOLS/YAML/YAML_Core/Quoting_and_Scalars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To comment out an entire block of YAML during debugging, you can use the # symbol to start a comment line in YAML. When you want to comment out multiple lines or a block, you simply add a # at the beginning of each line within that block. This allows you to temporarily disable those sections without permanently altering your configuration.

For example, if you have this block of YAML:

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: nginx
          image: nginx:latest

To comment out the entire Deployment block, you would modify it as follows:

YAML
# apiVersion: apps/v1
# kind: Deployment
# metadata:
#   name: nginx-deployment
# spec:
#   replicas: 3
#   template:
#     spec:
#       containers:
#         - name: nginx
#           image: nginx:latest

