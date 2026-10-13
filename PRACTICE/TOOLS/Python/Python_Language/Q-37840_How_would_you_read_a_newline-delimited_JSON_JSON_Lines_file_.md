---
type: "practice"
difficulty: "Basic"
---

# How would you read a newline-delimited JSON (JSON Lines) file in Python line by line, and why is this format friendlier for streaming than a single JSON array?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Parsing JSON Safely](<../../../../TOOLS/Python/Python_Language/Parsing_JSON_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To read a newline-delimited JSON (NDJSON) file in Python line by line, we need to understand that NDJSON is a simple text-based format where each line contains a valid JSON object. This makes it particularly friendly for streaming data processing because you can process one line at a time without loading the entire file into memory.

The core idea here is to use Python's built-in open function and iterate over the file line by line, parsing each line as a separate JSON object using a library like json. This approach ensures that we only work with small chunks of data at any given moment, making it efficient for large datasets.

**Code:**

```python
import json

def read_ndjson(file_path):
    """Read an NDJSON file line by line."""
    with open(file_path, 'r') as file:
        for line in file:
            # Parse the JSON object from each line
            data = json.loads(line)
            # Process your data here
            print(data)

# Example usage
read_ndjson('path/to/your/file.ndjson')
```

**Example input / output:**

```

Example input:

NDJSON
Copy
{"name": "Alice", "age": 30}
{"name": "Bob", "age": 25}
{"name": "Charlie", "age": 35}

Output:

CODE
Copy
{'name': 'Alice', 'age': 30}
{'name': 'Bob', 'age': 25}
{'name': 'Charlie', 'age': 35}
```

