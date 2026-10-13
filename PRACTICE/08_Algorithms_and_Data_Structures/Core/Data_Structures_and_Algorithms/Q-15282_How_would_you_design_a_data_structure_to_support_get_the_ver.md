---
type: "practice"
difficulty: "Expert"
---

# How would you design a data structure to support 'get the version of this document as of timestamp T' for a document that changes frequently, keeping storage and read cost reasonable?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Diff and LCS Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Diff_and_LCS_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of retrieving the version of a document as of a specific timestamp (T), we need to design a data structure that efficiently supports both write (document changes) and read operations while keeping storage and read costs reasonable. The core idea is to maintain an efficient log or history of all versions of the document, allowing quick lookups based on timestamps.

#### Triaging the Question

The question is probing how you would balance performance and resource utilization in a system where documents are frequently updated but must be retrievable at specific historical points. Key assumptions include:

The system supports frequent writes (document updates).
There's a need to query document states at arbitrary past timestamps.
The storage and read costs should be optimized.

#### First Principles Solution

We can design this using a combination of a log structure and efficient lookup mechanisms. Here are the steps:

Log Structure: Store each version of the document along with its timestamp in a sorted data structure, such as a balanced binary search tree (e.g., AVL Tree or Red-Black Tree).
Efficient Timestamp Lookup: Use a hash map to store the mapping from timestamps to document versions for quick access.
Binary Search Optimization: Perform binary search on the log to find the closest version before the requested timestamp (T).
Detailed Solution

Log Structure:

Store each version of the document along with its timestamp in a data structure that supports efficient insertion and lookup operations, such as a balanced BST.
This ensures that we can quickly insert new versions while maintaining order by time.

Hash Map for Quick Access:

Maintain an additional hash map where keys are timestamps and values are pointers to the corresponding document version in the log.
This allows quick lookup of any specific timestamp (T).

Binary Search:

When a request comes in for a specific timestamp (T), use binary search on the log to find the highest timestamp that is less than or equal to (T).
Return the corresponding document version.

#### Implementation
PYTHON
from bisect import bisect_left

class DocumentHistory:
    def __init__(self):
        self.log = []  # Stores [timestamp, documentVersion]
        self.timestamp_map = {}  # Maps timestamp to index in log

    def insert(self, timestamp, document_version):
        if timestamp not in self.timestamp_map:
            idx = bisect_left([t[0] for t in self.log], timestamp)
            self.log.insert(idx, [timestamp, document_version])
            self.timestamp_map[timestamp] = len(self.log) - 1

    def get_document_as_of(self, target_timestamp):
        if not self.log or target_timestamp < self.log[0][0]:
            return None
        idx = bisect_left([t[0] for t in self.log], target_timestamp)
        if idx == 0:
            return self.log[idx][1]
        else:
            return self.log[idx - 1][1]

#### Closing Summary
and Common Mistakes

To summarize, the key steps are to use a balanced BST for efficient log storage, a hash map for quick timestamp lookups, and binary search to find the relevant version. Common mistakes include not considering the need for balanced trees in frequent insertions or ignoring the importance of maintaining lookup efficiency.

#### Follow-Ups
How would you handle large-scale data?
What if documents are very large? Would your approach change?
How do you ensure consistency and performance under high concurrency?

**Comparison:**

```
Timestamp	Version
1	V1
3	V2
5	V3
```

**Code:**

```python
from bisect import bisect_left

class DocumentHistory:
    def __init__(self):
        self.log = []  # Stores [timestamp, documentVersion]
        self.timestamp_map = {}  # Maps timestamp to index in log

    def insert(self, timestamp, document_version):
        if timestamp not in self.timestamp_map:
            idx = bisect_left([t[0] for t in self.log], timestamp)
            self.log.insert(idx, [timestamp, document_version])
            self.timestamp_map[timestamp] = len(self.log) - 1

    def get_document_as_of(self, target_timestamp):
        if not self.log or target_timestamp < self.log[0][0]:
            return None
        idx = bisect_left([t[0] for t in self.log], target_timestamp)
        if idx == 0:
            return self.log[idx][1]
        else:
            return self.log[idx - 1][1]
```

**Example input / output:**

```text
Input:
DocumentHistory obj = new DocumentHistory();
obj.insert(1, "V1");
obj.insert(3, "V2");
obj.insert(5, "V3");

Output for get_document_as_of(4):
"V2"
```

**Data:**

```json
{
    "log": [
        {"timestamp": 1, "version": "V1"},
        {"timestamp": 3, "version": "V2"},
        {"timestamp": 5, "version": "V3"}
    ],
    "timestamp_map": {
        "1": 0,
        "3": 1,
        "5": 2
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Document History
5. Insert Document?
6. Update Log and TimestampMap
7. Retrieve Document as of T?
8. Binary Search for ClosestTimestamp < T
9. Return Corresponding Versionfrom Log

