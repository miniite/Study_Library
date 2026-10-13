---
type: "practice"
difficulty: "Intermediate"
---

# Design a versioned document/wiki with revision history. How do you model a diff and rollback?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Maintaining Denormalized Views](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/Maintaining_Denormalized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a versioned document/wiki system that tracks revisions, we need to focus on modeling both the documents themselves and their revisions, as well as implementing efficient ways to apply diffs and rollbacks. The core idea is to ensure that each revision retains enough context to fully reconstruct the state of the document at any given point in time.

Firstly, let’s break down what the question is really asking:

We need a system where documents can be versioned.
Each version should have its own revision history.
Efficient diffing and rolling back to previous versions must be supported.
Step-by-Step Solution
Document Model:
Store the document content in a structured way, e.g., as text or JSON.
Include metadata such as author, timestamp, and version number.
Revision History:
Maintain a log of all changes made to the document, including what changed and by whom.
Diffing Algorithm:
Implement an algorithm that can compute the differences between two versions of the document.
Rollback Mechanism:
Provide functionality to revert the document state to any previous version.
Detailed Breakdown

Document Model

SQL
CREATE TABLE documents (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    author_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

Revision History

SQL
CREATE TABLE revisions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    document_id INT NOT NULL,
    version_number INT,
    content TEXT,
    author_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (document_id) REFERENCES documents(id)
);

Diffing Algorithm

We can use a diff library like python-diff or xdelta for Python.
PYTHON
def compute_diff(old_content, new_content):
    # Use a library to generate differences
    from difflib import SequenceMatcher

    # Example using SequenceMatcher
    sm = SequenceMatcher(None, old_content, new_content)
    return sm.get_opcodes()

Rollback Mechanism

To rollback, we simply query the revisions table and apply them in reverse order.
SQL
-- Query for the previous version based on version_number
SELECT content FROM revisions WHERE document_id = <document_id> AND version_number = <previous_version>;

-- Apply the differences to the current state
UPDATE documents SET content = new_content;

#### Closing Summary
and Common Mistakes
Ensure that the diffing algorithm is efficient, especially for large documents.
Handle edge cases like conflicts during simultaneous edits or inconsistent states in the database.

#### Cons
ider storage optimizations such as compressing diffs to save space.

#### Example
Worked Input->Output
PLAINTEXT
Initial Document Content: "This is a test document."
Updated Document Content: "This updated test document has new content."

# Compute Diff
Diff Opcodes:
[('equal', 0, 27, 0, 26),
 ('delete', 27, 52, None, -1)]

# Rollback to Initial State
Rollback Command:
SELECT content FROM revisions WHERE document_id = <doc_id> AND version_number = <version_before_update>;

**Comparison:**

| Document ID | Title     | Content            | Author ID | Created At  | Updated At           |
|------------|-----------|--------------------|----------|-------------|---------------------|
| 1          | Test Doc  | This is a test doc | 1        | 2023-05-01  | 2023-05-01 14:30:00 |
| 2          | Updated   | This updated doc   | 2        | 2023-05-02  | 2023-05-02 15:00:00 |
| Revision ID | Document ID | Version Number | Content                  | Author ID | Created At  |
|-------------|------------|----------------|--------------------------|-----------|-------------|
| 1           | 1          | 1              | This is a test doc       | 1         | 2023-05-01  |
| 2           | 1          | 2              | This updated test doc   | 2         | 2023-05-02  |

**Code:**

```python
from difflib import SequenceMatcher

def compute_diff(old_content, new_content):
    sm = SequenceMatcher(None, old_content, new_content)
    return sm.get_opcodes()
```

**Example input / output:**

```text
Initial Document Content: "This is a test document."
Updated Document Content: "This updated test document has new content."

# Compute Diff
Diff Opcodes:
[('equal', 0, 27, 0, 26),
 ('delete', 27, 52, None, -1)]

# Rollback to Initial State
Rollback Command:
SELECT content FROM revisions WHERE document_id = <doc_id> AND version_number = <version_before_update>;
```

**Data:**

```sql
CREATE TABLE documents (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    author_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE revisions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    document_id INT NOT NULL,
    version_number INT,
    content TEXT,
    author_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (document_id) REFERENCES documents(id)
);
```

**Flow:**

1. Yes
2. No
3. User Updates Document
4. Compute Diff
5. Store New Version
6. Rollback Request?
7. Retrieve Previous Revision
8. Apply Revisions to CurrentState
9. Document Is Latest Version

