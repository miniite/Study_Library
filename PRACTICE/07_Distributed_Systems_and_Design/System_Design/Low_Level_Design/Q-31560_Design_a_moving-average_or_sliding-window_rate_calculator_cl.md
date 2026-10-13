---
type: "practice"
difficulty: "Deep"
---

# Design a moving-average or sliding-window rate calculator class. How do you evict old data points?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Maintaining Denormalized Views](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/Maintaining_Denormalized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a moving-average or sliding-window rate calculator, we need to focus on maintaining a window of recent data points while efficiently removing older ones as new data comes in. This is crucial to ensure that our calculations remain accurate and up-to-date without consuming excessive memory resources.

#### Triaging the Question

The question is asking how to manage an efficient mechanism for evicting old data points from the sliding window. The interviewer is probing your understanding of data structures, time complexity considerations, and optimization techniques. We need to make some assumptions: the rate at which new data comes in, the size of the sliding window, and whether we're dealing with timestamps or intervals.

#### First-Principles Solution
Data Structure Selection: Use a deque (double-ended queue) from Python's collections module. A deque allows efficient insertion and removal from both ends.
Timestamp Handling: Ensure each data point has an associated timestamp to maintain order as new data comes in.
Eviction Mechanism: When the window exceeds its size, remove the oldest entry first.

Here’s a step-by-step breakdown:

Initialize a deque to hold the data points and their timestamps.
As new data arrives, add it along with its timestamp to the end of the deque.
If the deque reaches the maximum size defined by the sliding window, remove the oldest element (from the front) to keep the window within bounds.

#### Implementation
PYTHON
from collections import deque

class SlidingWindowRateCalculator:
    def __init__(self, max_window_size):
        self.max_window_size = max_window_size
        self.window = deque()

    def add_data_point(self, timestamp, value):
        # Remove old data points if the window is full
        while len(self.window) >= self.max_window_size and timestamp - self.window[0][0] > 1:
            self.window.popleft()

        # Add new data point
        self.window.append((timestamp, value))

    def calculate_average(self):
        total = sum(value for _, value in self.window)
        return total / len(self.window) if self.window else None

# Example Usage
calculator = SlidingWindowRateCalculator(max_window_size=10)
timestamps = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
values = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 110]

for timestamp, value in zip(timestamps, values):
    calculator.add_data_point(timestamp, value)
    print(f"Current average: {calculator.calculate_average()}")

#### Closing Summary

By using a deque to manage the sliding window, we ensure that adding and removing elements is efficient. The add_data_point method handles both adding new data points and evicting old ones based on the current timestamp. This approach maintains an optimal balance between memory usage and computational efficiency.

#### Common Mistakes/Follow-ups
Not Considering Time Delta: Failing to check if the time difference is within the window’s constraints.
Improper Deque Management: Not correctly managing the deque, leading to either too many or too few evictions.
Incorrect Calculation of Average: Miscomputing the average due to incorrect handling of data points.

