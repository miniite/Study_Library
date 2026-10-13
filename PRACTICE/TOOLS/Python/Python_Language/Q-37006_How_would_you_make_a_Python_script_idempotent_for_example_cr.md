---
type: "practice"
difficulty: "Basic"
---

# How would you make a Python script idempotent, for example creating a directory only if it does not already exist?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Idempotent Event Handlers](<../../../../TOOLS/Python/Python_Language/Idempotent_Event_Handlers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To make a Python script idempotent, such as ensuring that a directory is created only if it doesn't already exist, we need to ensure that running the script multiple times won't result in duplicate operations or unintended side effects. The key idea here is to check for the existence of the directory before creating it and proceed accordingly.

The core concept involves using Python's os module to interact with the file system. We will use the os.path.exists function to check if the directory already exists, and if not, we'll create it using os.makedirs.

Here’s a step-by-step breakdown of how to implement this:

Check for Directory Existence: Use os.path.exists(directory_path) to determine if the directory already exists.
Create Directory If Not Exists: If the directory does not exist, use os.makedirs(directory_path) to create it.

Let's write out a simple Python script that accomplishes this:

PYTHON
import os

def ensure_directory_exists(directory_path):
    # Check if the directory already exists
    if not os.path.exists(directory_path):
        # Create the directory if it does not exist
        os.makedirs(directory_path)
        print(f"Directory {directory_path} created.")
    else:
        print(f"Directory {directory_path} already exists.")

# Example usage
ensure_directory_exists("/path/to/directory")

This script ensures that the specified directory is only created once, making it idempotent. If you run this script multiple times with the same directory_path, no additional directories will be created.

**Code:**

```python
import os

def ensure_directory_exists(directory_path):
    if not os.path.exists(directory_path):
        os.makedirs(directory_path)
        print(f"Directory {directory_path} created.")
    else:
        print(f"Directory {directory_path} already exists.")

# Example usage
ensure_directory_exists("/path/to/directory")
```

**Example input / output:**

```

Example input and output:

PLAINTEXT
Copy
Input: ensure_directory_exists("/tmp/mydir")

Output:
Directory /tmp/mydir created.

If the directory already exists, it will print a message indicating that it already exists.

PLAINTEXT
Copy
Input: ensure_directory_exists("/tmp/mydir")

Output:
Directory /tmp/mydir already exists.
```

