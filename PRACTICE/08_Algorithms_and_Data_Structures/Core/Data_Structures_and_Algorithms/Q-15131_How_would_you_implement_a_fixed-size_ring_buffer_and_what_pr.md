---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a fixed-size ring buffer and what problem does it solve?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stream Join Data Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Stream_Join_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

A fixed-size ring buffer is an efficient data structure that stores a fixed number of elements, overwriting the oldest element when new ones are added. It’s particularly useful in scenarios where you need to maintain a history of recent events or values without consuming unbounded memory.

#### Triage

The question asks for the implementation details of a fixed-size ring buffer and its application. The key aspects include understanding how to manage memory efficiently, dealing with edge cases like overflow, and ensuring that data is accessed in the correct order.

Step-by-Step Solution:

Understanding the Problem:

A fixed-size ring buffer has a predefined capacity.
It uses a circular array where elements are written at the "tail" and read from the "head."
When the tail reaches the end of the buffer, it wraps around to the beginning.

Key Components:

buffer: An array of fixed size.
capacity: The maximum number of elements that can be stored.
head: Pointer to the current read position.
tail: Pointer to the next write position.

#### Implementation
Details:

PYTHON
class RingBuffer:
    def __init__(self, capacity):
        self.capacity = capacity
        self.buffer = [None] * capacity
        self.head = 0  # Next position to read from
        self.tail = 0  # Current write position
        self.size = 0  # Number of elements currently in buffer

    def append(self, value):
        if self.size == self.capacity:
            self.head = (self.head + 1) % self.capacity  # Overflow, wrap around
        else:
            self.size += 1
        self.buffer[self.tail] = value
        self.tail = (self.tail + 1) % self.capacity

    def get(self):
        if self.size == 0:
            return None
        val = self.buffer[self.head]
        self.head = (self.head + 1) % self.capacity
        self.size -= 1
        return val

Edge Cases and Performance Considerations:

Overflow: When the buffer is full, new writes should overwrite the oldest data.
Underflow: In practice, this isn't a problem since writes always occur after reads.
Memory Efficiency: The buffer uses minimal memory as it only stores elements that are currently relevant.

Accessing Elements:

To access an element at position i, use the formula (head + i) % capacity.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Mistakes to Avoid:

Forgetting to wrap around when the buffer is full.
Not incrementing the head or tail correctly.
Accessing elements out of bounds.

#### Follow-Ups

How would you implement a ring buffer with a dynamic capacity?
What are some use cases for a ring buffer in real-world applications?

