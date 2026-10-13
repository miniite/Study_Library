---
type: "practice"
difficulty: "Basic"
---

# How do you handle a FileNotFoundError gracefully in a Python script that reads a config file that might not exist yet?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Subprocess Safety](<../../../../TOOLS/Python/Python_Language/Subprocess_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When writing a Python script to read a configuration file, it's essential to handle potential issues like FileNotFoundError gracefully. This ensures your application can run smoothly even if the expected file is missing or hasn't been created yet. Let's walk through how you would implement this from first principles.

First, we need to understand that when a Python script attempts to open a non-existent file using built-in functions like open(), it raises a FileNotFoundError. The task here is to catch this exception and handle it appropriately without crashing the program or causing unexpected behavior. We'll cover the steps in detail.

**Code:**

```python
try:
    with open('config.yaml', 'r') as file:
        config_data = yaml.safe_load(file)
except FileNotFoundError:
    print("Configuration file not found. Initializing default values.")
    # Initialize default configuration here
```

**Example input / output:**

```text
# Example input and expected output
try:
    with open('config.yaml', 'r') as file:
        config_data = yaml.safe_load(file)
except FileNotFoundError:
    print("Configuration file not found. Initializing default values.")
    # Initialize default configuration here

# Expected output
print(config_data)  # Output depends on the initialization of default values
```

**Flow:**

1. Yes
2. N
3. Start
4. Attempt to open config file
5. File exists?
6. Read and process file data
7. Initialize default values
8. Output processed data or usedefaults
9. End

