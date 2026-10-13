---
type: "practice"
difficulty: "Deep"
---

# What is pickling, what are its security risks, and why should you never unpickle untrusted data?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prompt Injection Defense](<../../../../TOOLS/Python/Python_Language/Prompt_Injection_Defense.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Pickling in Python refers to the process of converting a Python object into a byte stream, which can be stored or transmitted, and later reconstructed back into an equivalent object. This is particularly useful for serializing complex data structures like dictionaries or classes. However, unpickling untrusted data poses significant security risks, as it can execute arbitrary code in your application.

#### Triage
What the Question Is Really Asking

The question delves into understanding how pickling works and why handling untrusted pickled data is dangerous. The interviewer is probing your knowledge of Python's serialization mechanisms, potential security vulnerabilities, and safe coding practices.

#### First-Principles Solution

#### Overview
of Pickling

Pickling in Python allows you to serialize an object graph (a collection of interconnected objects) into a byte stream that can be stored or transmitted over a network. The process involves converting the object's state into a format that can be later deserialized back into its original form. Here’s how it works:

PYTHON
import pickle

# Example pickling and unpickling
data = {'a': [1, 2, 3], 'b': ('foo', 'bar')}
with open('data.pickle', 'wb') as f:
    pickle.dump(data, f)

with open('data.pickle', 'rb') as f:
    loaded_data = pickle.load(f)
Security Risks of Unpickling

Unpickling untrusted data can be extremely dangerous because the process involves executing arbitrary code. When you unpickle a byte stream, Python executes the code embedded within it, which means an attacker could inject malicious code that will run in your application context.

Here’s a concrete example:

PYTHON
import pickle

def execute_malicious_code():
    import os
    os.system("rm -rf /")

with open('malicious.pickle', 'wb') as f:
    pickle.dump(execute_malicious_code, f)

# If this file is provided by an untrusted source and unpickled, it will execute the function.
with open('malicious.pickle', 'rb') as f:
    malicious_function = pickle.load(f)
    malicious_function()  # This line will delete all files on the system!

#### Closing Summary
+ Common Mistakes/Follow-Ups

To avoid such security risks, you should never unpickle data that comes from untrusted sources. Always validate and sanitize your input before unpickling. Additionally, consider using alternative serialization formats like JSON for public-facing applications where security is paramount.

#### Common Mistakes
Unpickling Untrusted Data: Accidentally handling untrusted pickled data can lead to code injection.
Overlooking Security Checks: Not implementing robust checks to verify the integrity and trustworthiness of the pickle data.
Misunderstanding Pickle's Execution Power: Thinking that pickling is just about saving state when it also has powerful execution capabilities.

#### Follow-Ups
How do you ensure the security of pickled data in your applications?
Are there any alternatives to using pickle for serialization in Python?
Can you discuss the trade-offs between using pickle and other serialization methods like JSON or Protocol Buffers?

**Comparison:**

```
Step	Action
1	Pickle the dictionary.
2	Write pickled data to a file.
3	Read pickled data from the file.
4	Unpickle and load the dictionary back into memory.
```

**Code:**

```python
import pickle

# Example pickling and unpickling
data = {'a': [1, 2, 3], 'b': ('foo', 'bar')}
with open('data.pickle', 'wb') as f:
    pickle.dump(data, f)

with open('data.pickle', 'rb') as f:
    loaded_data = pickle.load(f)
```

**Example input / output:**

```text
Given the input `{'a': [1, 2, 3], 'b': ['foo', 'bar']}`:

- The pickled data is written to a file named `data.pickle`.
- When unpickling this file, it correctly loads the dictionary back into memory.
```

**Data:**

```json
{
  "a": [1, 2, 3],
  "b": ["foo", "bar"]
}
```

**Flow:**

1. Yes
2. No
3. Pickling
4. Is Data Trusted?
5. Pickle to File
6. Read Pickle from File
7. Unpickle and Load
8. Handle with Caution

