---
type: "practice"
difficulty: "Intermediate"
---

# How would you capture both stdout and stderr from a shell command in Python and handle them separately?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Subprocess Safety](<../../../../TOOLS/Python/Python_Language/Subprocess_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To capture both stdout and stderr from a shell command in Python, we need to use the subprocess module, which provides a powerful interface for spawning new processes, connecting to their input/output/error pipes, and obtaining their return codes. The key idea is that we can capture both outputs separately by using separate file descriptors for stdout and stderr.

**Code:**

```python
import subprocess

# Example command: 'echo "Hello" 1>&2' which sends the output to stderr
command = ['echo', '"Hello"', '1>&2']

# Using subprocess.Popen to capture both stdout and stderr separately
process = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE)

stdout_output, stderr_output = process.communicate()

print("Stdout Output:", stdout_output.decode())
print("Stderr Output:", stderr_output.decode())
```

**Example input / output:**

```text
Stdout Output:
(b'')
Stderr Output:
b'Hello\n'
```

