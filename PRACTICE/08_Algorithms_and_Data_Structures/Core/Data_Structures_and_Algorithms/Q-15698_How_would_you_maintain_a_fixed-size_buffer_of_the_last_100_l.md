---
type: "practice"
difficulty: "Intermediate"
---

# How would you maintain a fixed-size buffer of the last 100 latency measurements from LLM calls to compute a rolling average, and which structure gives O(1) push and pop?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Sliding Window Aggregates](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To maintain a fixed-size buffer of the last 100 latency measurements from language model (LLM) calls while computing a rolling average with O(1) time complexity for both insertions and deletions, we need to use a data structure that efficiently supports these operations. The ideal choice here is a circular buffer or ring buffer, which allows us to maintain a fixed-size collection in a way that insertion at the end and deletion from the front can be performed in constant time.

#### Triage

The question asks for a method to store the last 100 latency measurements efficiently. The key points are:

The data structure should support O(1) insertions and deletions.
It must maintain only the most recent 100 measurements.
We need to calculate a rolling average from these measurements.

#### Solution

A circular buffer works perfectly for this scenario because it has two main operations: push (inserting an element at the end of the buffer) and pop (removing an element from the front of the buffer). Both operations can be performed in constant time, O(1).

Step-by-Step Solution
Initialize the Buffer: Create a fixed-size array of size 100 to store the latency measurements.
Circular Indexing: Use two pointers or indices to track the current read and write positions within the buffer. These can wrap around when they reach the end, hence the name "circular".
Push Operation: When inserting a new measurement:
Place it in the write position.
Update the write pointer.
Pop Operation: When computing or removing an old measurement:
Take the value from the read position.
Increment (or wrap) the read pointer.

Here's how this works:

To push a latency measurement, place it at the current write index and increment write.
If write reaches the end of the buffer, reset it to 0.
To pop an old measurement for computing the average, take its value from the current read index and increment (or wrap) the read.

#### Example
Code

Let's implement this in Python:

PYTHON
class RollingBuffer:
    def __init__(self, size=100):
        self.size = size
        self.buffer = [None] * size  # Fixed-size buffer
        self.write_idx = 0          # Write index for new elements
        self.read_idx = 0           # Read index for oldest elements

    def push(self, value):
        """Push a new latency measurement into the buffer."""
        self.buffer[self.write_idx % self.size] = value
        self.write_idx += 1

    def pop(self):
        """Return and remove the oldest latency measurement from the buffer."""
        if not self.is_empty():
            val = self.buffer[self.read_idx % self.size]
            self.read_idx += 1
            return val
        return None

    def is_empty(self):
        """Check if the buffer is empty."""
        return self.write_idx == self.read_idx

    def average(self):
        """Compute and return the rolling average of all measurements in the buffer."""
        total = sum(self.buffer[self.read_idx % self.size: self.write_idx % self.size])
        count = min(self.write_idx - self.read_idx, self.size)
        if count == 0:
            return 0
        return total / count

# Example usage
buffer = RollingBuffer(100)
for i in range(101):
    buffer.push(i * 10)  # Simulate LLM latency measurements
print(buffer.average())  # Should print the average of the last 100 values

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, a circular buffer is the most efficient data structure for maintaining a rolling average with constant-time operations. This approach ensures that we can quickly insert and remove measurements while keeping the computational overhead minimal.

#### Common mistakes
include:

Using a non-circular list or array, which would require shifting all elements on each insertion.
Forgetting to wrap indices when they reach the end of the buffer.

#### Follow-up
questions might explore different scenarios, such as handling larger datasets (e.g., 10,000 measurements), or considering performance implications in edge cases like continuous high-rate insertions and deletions.

