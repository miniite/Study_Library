---
type: "practice"
difficulty: "Basic"
---

# How would you write a Python script that reads a list of server hostnames from a file and prints each one, one per line?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Subprocess Safety](<../../../../TOOLS/Python/Python_Language/Subprocess_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to create a simple Python script that reads a text file containing a list of server hostnames and prints each hostname on a new line. This is a fundamental task in DevOps that involves basic file handling and loop operations.

First, let's clarify the assumptions:

The input file contains one hostname per line.
Each hostname should be printed exactly as it appears in the file, with no modifications.
We'll assume the file is named hostnames.txt for simplicity.

Now, we will write a Python script that performs these tasks:

Open the file in read mode.
Read each line from the file.
Print each line to the console.

Here’s how you can implement this:

PYTHON
# Open the file and read its content
with open('hostnames.txt', 'r') as file:
    # Iterate over each line in the file
    for hostname in file:
        # Strip any extra whitespace (like newline character) and print it
        print(hostname.strip())

To ensure our script works correctly, we can create a sample hostnames.txt file with some example data:

PLAINTEXT
server1.example.com
server2.example.com
server3.example.com

Running the Python script on this input should produce:

CODE
server1.example.com
server2.example.com
server3.example.com

**Code:**

```python
# Open the file and read its content
with open('hostnames.txt', 'r') as file:
    # Iterate over each line in the file
    for hostname in file:
        # Strip any extra whitespace (like newline character) and print it
        print(hostname.strip())
```

**Example input / output:**

```

Input hostnames.txt:

CODE
Copy
server1.example.com
server2.example.com
server3.example.com

Output:

CODE
Copy
server1.example.com
server2.example.com
server3.example.com
```

**Data:**

```plaintext
server1.example.com
server2.example.com
server3.example.com
```

**Flow:**

1. Yes
2. Yes
3. No
4. Read file
5. Is file open?
6. Iterate over each line
7. Print line
8. Move to next line
9. Close file

