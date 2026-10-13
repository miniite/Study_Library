---
type: "practice"
difficulty: "Basic"
---

# What is the file extension convention for YAML files and does the tool care whether it is .yml or .yaml?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ YAML and JSON](<../../../../TOOLS/YAML/YAML_Core/YAML_and_JSON.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with YAML files, the most common file extensions are .yml and .yaml. These extensions are used interchangeably to denote a file that contains data in YAML format. The tool (parser) does not inherently care about the specific extension as long as the content within the file adheres to the syntax rules of YAML.

Let's break down the core aspects:

File Extension Conventions:

.yml: This is more commonly used and has a shorter name, making it easier to type.
.yaml: This is also valid and was historically used because "YAML" stands for "YAML Ain't Markup Language," which can be seen as "YAML Ain't Markup Language".

Tool Behavior:

Most YAML parsers and tools (like pyyaml in Python, or yq for query) will recognize both extensions.
The tool checks the content of the file to ensure it follows the correct syntax rules, not specifically which extension is used.

Why Both Exist?:

.yml: Simplified version intended for quick usage and easier typing.
.yaml: Formalized version that keeps with the language's naming conventions but is less commonly seen in practice.

In summary, while both extensions are valid, using .yml or .yaml will work as long as the content adheres to YAML syntax. The tool does not differentiate based on the extension; it focuses on the data structure and its validity.

**Comparison:**

```
File Extension	Common Usage
.yml	Quick and common
.yaml	Formal but less used
```

**Code:**

```python
# Example of reading a YAML file with both extensions in Python
import yaml

def read_yaml(file_path):
    try:
        with open(file_path, 'r') as file:
            return yaml.safe_load(file)
    except FileNotFoundError:
        print(f"File not found: {file_path}")
    except Exception as e:
        print(f"Error reading YAML file: {e}")

# Example usage
data1 = read_yaml('example.yml')
data2 = read_yaml('example.yaml')

print(data1 == data2)  # This should output True, indicating both work equally well
```

**Example input / output:**

```

Input (file example.yml):

YAML
Copy
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: busybox
      command: ['sh', '-c', 'echo Hello world; sleep 3600']

Output:

PYTHON
Copy
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: busybox
      command: ['sh', '-c', 'echo Hello world; sleep 3600']

Input (file example.yaml):

YAML
Copy
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: busybox
      command: ['sh', '-c', 'echo Hello world; sleep 3600']

Output:

PYTHON
Copy
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: busybox
      command: ['sh', '-c', 'echo Hello world; sleep 3600']
```

**Data:**

```yaml
# Example of a simple YAML file
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: busybox
      command: ['sh', '-c', 'echo Hello world; sleep 3600']
```

**Flow:**

1. .yml or .yaml
2. Read YAML File
3. Is the extension '.yml' or'.yaml'?
4. Load file content
5. Parsing and validation of YAMLsyntax
6. Return parsed data

