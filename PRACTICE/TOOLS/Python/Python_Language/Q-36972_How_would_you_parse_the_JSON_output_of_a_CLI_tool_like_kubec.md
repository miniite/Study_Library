---
type: "practice"
difficulty: "Intermediate"
---

# How would you parse the JSON output of a CLI tool like 'kubectl get pods -o json' or 'docker inspect' inside a Python script?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Parsing JSON Safely](<../../../../TOOLS/Python/Python_Language/Parsing_JSON_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To parse the JSON output from tools like kubectl or docker inspect within a Python script, you first need to understand that these commands return structured data in JSON format. Your task is essentially about extracting and manipulating this data using Python's powerful handling capabilities for JSON. Here’s how you can approach it step-by-step:

Firstly, the core idea is to capture the output from the CLI tool as a string, then convert that string into a Python dictionary or list of dictionaries. Once in a Pythonic format, you can easily access and manipulate the data using various methods provided by Python's json module.

Now let's dive into the solution:

#### Triaging the Question

The question asks us to parse JSON output from CLI tools like kubectl get pods -o json or docker inspect. The key points are understanding how to capture the raw output, convert it to a Python data structure, and then work with that data.

#### Assumptions
You are running these commands in a Unix-based environment.
You have basic knowledge of Python programming.
You want to extract specific fields from the JSON data for further processing.

#### First-Principles Solution
Run the CLI Command: Use subprocess or os module to run the command and capture its output.
Convert Output to Python Data Structure: Use json.loads() method to convert the string output into a Python dictionary or list of dictionaries.
Access and Manipulate Data: Once in Python, you can use various methods like dictionary indexing, loops, etc., to access and manipulate the data.

#### Example
Code
PYTHON
import subprocess
import json

def get_kubectl_pods():
    # Run kubectl command and capture output
    result = subprocess.run(['kubectl', 'get', 'pods', '-o', 'json'], stdout=subprocess.PIPE)

    # Convert JSON string to Python dictionary
    pods_data = json.loads(result.stdout.decode('utf-8'))

    return pods_data

# Example usage
pods_info = get_kubectl_pods()
for pod in pods_info['items']:
    print(f"Pod Name: {pod['metadata']['name']}, Status: {pod['status']['phase']}")

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key steps involve running the CLI command to capture its output as a string, converting that string into Python data structures using json.loads(), and then processing it. Common mistakes include not handling errors (e.g., command failure) or not correctly decoding the JSON string.

For follow-ups, you might be asked about dealing with large datasets efficiently, managing different JSON formats, or integrating this into a larger pipeline or application.

**Code:**

```python
import subprocess
import json

def get_kubectl_pods():
    result = subprocess.run(['kubectl', 'get', 'pods', '-o', 'json'], stdout=subprocess.PIPE)
    pods_data = json.loads(result.stdout.decode('utf-8'))
    return pods_data

# Example usage
pods_info = get_kubectl_pods()
for pod in pods_info['items']:
    print(f"Pod Name: {pod['metadata']['name']}, Status: {pod['status']['phase']}")
```

**Example input / output:**

```text
$ kubectl get pods -o json | python parse_kubectl.py
Pod Name: pod1, Status: Running
Pod Name: pod2, Status: Running
```

