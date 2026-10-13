---
type: "practice"
difficulty: "Intermediate"
---

# How do you list all files in a directory recursively and filter by extension using pathlib or os.walk?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Compressed File Handling](<../../../../TOOLS/Python/Python_Language/Compressed_File_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To list all files in a directory recursively and filter by extension, we can use Python's pathlib module or os.walk(). Both approaches are common and effective, but pathlib is generally more concise and readable. We'll go through both methods from first principles.

**Code:**

```

from pathlib import Path

def list_files_recursively(path, extensions=None): """List all files in a directory recursively with optional extension filtering.""" # Walk through the path and filter by file extensions if provided. for p in Path(path).rglob('*'): if p.is_file(): if extensions is None or p.suffix.lower() in extensions: print(p)

CODE
Copy

```python
import os

def list_files_recursively_os_walk(path, extensions=None):
    """List all files in a directory recursively with optional extension filtering using os.walk()."""
    # Walk through the path and filter by file extensions if provided.
    for root, dirs, files in os.walk(path):
        for file in files:
            p = os.path.join(root, file)
            if extensions is None or p.endswith(tuple(extensions)):
                print(p)
```

**Example input / output:**

```text
# Example usage of `list_files_recursively` with filtering by .txt and .md files:
list_files_recursively("/path/to/directory", ['.txt', '.md'])

Output:
/path/to/directory/file1.txt
/path/to/directory/nested/folder1/file2.md

# Example usage of `list_files_recursively_os_walk` with no extension filter:
list_files_recursively_os_walk("/path/to/directory")

Output:
/path/to/directory/file3.py
/path/to/directory/nested/folder2/subfile.txt
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Is path a directory?
7. Walk through directory
8. Check if item is file
9. Print file path
10. Continue walking or return toparent folder
11. End

