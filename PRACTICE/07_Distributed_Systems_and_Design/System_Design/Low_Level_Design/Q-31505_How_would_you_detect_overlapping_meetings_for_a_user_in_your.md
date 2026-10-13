---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect overlapping meetings for a user in your calendar design and what is the complexity?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumer Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect overlapping meetings for a user in a calendar system, we need to ensure that no two meeting slots for a specific user overlap with each other. This involves designing an efficient data structure and algorithm to check for such overlaps. At the backend SDE-2/3 level, you should be able to implement this solution using basic data structures like lists or trees, while at the Architect/Principal level, you might focus on scalability, performance trade-offs, and potential optimizations.

The core idea is to use a sorted list (or a balanced binary search tree) to store meeting intervals. This allows us to perform efficient searches and insertions in logarithmic time. Here's how we can approach this from first principles:

#### Triage
The interviewer is probing your understanding of data structures, algorithm design, and performance considerations for handling overlapping meetings.

#### Assumptions
Each meeting has a start time and end time.
Meetings are stored in an array or list, where each entry represents the interval [start_time, end_time].

#### First Principles Solution
Sort the meeting intervals based on their start times (this can be done using any efficient sorting algorithm like merge sort).
Iterate through the sorted meetings and check for overlaps by comparing the current meeting's end time with the next meeting's start time.
Complexity:
Sorting the meetings takes (O(N \log N)), where (N) is the number of meetings.
Checking for overlaps after sorting takes (O(N)).
The overall complexity is dominated by the sorting step, making it (O(N \log N)).

Now let's walk through an example to better understand this approach.

**Code:**

```python
def detect_overlaps(meetings):
    # Step 1: Sort meetings based on start time
    meetings.sort(key=lambda x: x[0])

    # Step 2: Check for overlaps
    n = len(meetings)
    for i in range(n - 1):
        if meetings[i][1] > meetings[i + 1][0]:
            return True  # Overlapping meeting found

    return False  # No overlapping meetings

# Example data
meetings = [
    [9, 12],  # [start_time, end_time]
    [13, 15],
    [16, 18],
    [14, 17]  # This one should overlap with the previous meeting
]

print(detect_overlaps(meetings))  # Output: True
```

**Flow:**

1. Yes
2. No
3. Sort meetings by start time
4. B
5. Are there more meetings?
6. Check for overlaps betweenconsecutive meetings
7. Return True if overlap found
8. Return False (no overlappingmeetings)
9. Continue checking remainingmeetings

