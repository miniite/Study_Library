---
type: "practice"
difficulty: "Intermediate"
---

# What is a rolling window in metric analysis, and how would you use Python collections.deque to keep only the last 10 readings for a live check?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Data Basics](<../../../../TOOLS/Python/Python_Language/Time-series_Data_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rolling windows are essential in time-series data analysis, where we need to analyze metrics over a sliding window of recent observations. For instance, if you're monitoring system performance every minute and want to see the average of the last 10 readings, a rolling window helps achieve this efficiently. In Python, collections.deque is a perfect choice for implementing such windows because it provides fast appends and pops from both ends.

To keep only the last 10 readings using collections.deque, we'll follow these steps:

Initialize the deque: Create a deque with a maximum length of 10.
Update the deque on each new reading: Append the new reading to the end, and if it exceeds 10 elements, remove the oldest element from the front.

Let's dive into how we can implement this in Python:

