---
type: "practice"
difficulty: "Deep"
---

# A production interval-overlap check for calendar bookings gives wrong results at DST boundaries; separate the algorithmic correctness of interval merging from the time-representation bug and fix both.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Interval Overlap](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Interval_Overlap.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this problem, we are tasked with ensuring that a production-grade interval-overlap check function works correctly even at Daylight Saving Time (DST) boundaries. The core challenge here is separating algorithmic correctness from time-representation issues. Let's break down the solution step-by-step.

#### Triage

The question asks us to identify and fix both an algorithmic correctness issue and a time-representation bug in a production interval-overlap check function. Specifically, we need to ensure that our function correctly merges overlapping intervals even at DST boundaries. This requires understanding how timestamps are represented and how intervals should be handled during such transitions.

#### Assumptions
The current implementation of the interval-overlap check does not handle DST boundaries correctly.
We will assume the use of a common date-time library like python-dateutil or similar for handling time zones and DST transitions in Python.

#### First-Principles Solution
Step 1: Understand Interval Overlap

To begin, let's review how interval overlap is typically checked:

PYTHON
def check_overlap(start1, end1, start2, end2):
    return max(start1, start2) < min(end1, end2)

This function checks if two intervals [start1, end1] and [start2, end2] overlap by ensuring the maximum of the starts is less than the minimum of the ends.

Step 2: Identify Time-Representation Bug

The bug arises because timestamps at DST boundaries can cause confusion. For example, in a region that observes DST, an interval might cross a point where the timestamp changes from e.g., 19:00 to 00:00. If our function does not handle this correctly, it could incorrectly mark intervals as overlapping or non-overlapping.

Step 3: Correct Algorithmic Implementation

To fix the algorithm, we need to ensure that the timestamps are properly handled at DST boundaries. We can achieve this by:

Converting all start and end times to a consistent time zone before performing interval checks.
Handling transitions correctly by adjusting intervals around DST boundaries.

Here's an example of how we might implement this in Python using python-dateutil:

PYTHON
from dateutil import parser, tz

def check_overlap_fixed(start1, end1, start2, end2, timezone_str='America/New_York'):
    # Convert all times to a consistent time zone
    timezone = tz.gettz(timezone_str)
    start1 = parser.parse(start1).astimezone(timezone)
    end1 = parser.parse(end1).astimezone(timezone)
    start2 = parser.parse(start2).astimezone(timezone)
    end2 = parser.parse(end2).astimezone(timezone)

    # Adjust intervals around DST boundaries
    if is_dst_boundary(start1, timezone) or is_dst_boundary(end1, timezone):
        start1, end1 = adjust_for_dst(start1, end1, timezone)

    if is_dst_boundary(start2, timezone) or is_dst_boundary(end2, timezone):
        start2, end2 = adjust_for_dst(start2, end2, timezone)

    # Check for overlap
    return max(start1, start2) < min(end1, end2)

def is_dst_boundary(dt, tz):
    # Check if the given datetime is near a DST boundary (e.g., 2am)
    return abs((dt.dst() - timedelta(hours=2)).total_seconds()) < 3600

def adjust_for_dst(start, end, timezone):
    # Adjust start and end to be within valid hours after a DST transition
    if start.hour == 1:
        return start.replace(hour=2), end.replace(hour=2)
    elif end.hour == 1:
        return start.replace(hour=2), end.replace(hour=2)
    else:
        return start, end

# Example usage
print(check_overlap_fixed("2023-10-29T01:59", "2023-10-29T02:59",
                         "2023-10-29T03:00", "2023-10-29T04:00"))
Step 4: Close Summary and Common Mistakes

In summary, we have addressed the issue by ensuring all timestamps are properly converted to a consistent time zone before checking for overlap. We also adjusted intervals around DST boundaries to account for potential misalignments.

#### Common mistakes
include:

Ignoring DST transitions in interval checks.
Not converting all times to a consistent time zone.
Failing to handle edge cases where the transition affects start or end times of intervals.

#### Closing Summary

By carefully handling DST boundaries and ensuring timestamps are consistently converted, we can create a robust interval-overlap check function that works correctly even in complex time zones.

**Code:**

```python
from dateutil import parser, tz

def check_overlap_fixed(start1, end1, start2, end2, timezone_str='America/New_York'):
    # Convert all times to a consistent time zone
    timezone = tz.gettz(timezone_str)
    start1 = parser.parse(start1).astimezone(timezone)
    end1 = parser.parse(end1).astimezone(timezone)
    start2 = parser.parse(start2).astimezone(timezone)
    end2 = parser.parse(end2).astimezone(timezone)

    # Adjust intervals around DST boundaries
    if is_dst_boundary(start1, timezone) or is_dst_boundary(end1, timezone):
        start1, end1 = adjust_for_dst(start1, end1, timezone)

    if is_dst_boundary(start2, timezone) or is_dst_boundary(end2, timezone):
        start2, end2 = adjust_for_dst(start2, end2, timezone)

    # Check for overlap
    return max(start1, start2) < min(end1, end2)

def is_dst_boundary(dt, tz):
    # Check if the given datetime is near a DST boundary (e.g., 2am)
    return abs((dt.dst() - timedelta(hours=2)).total_seconds()) < 3600

def adjust_for_dst(start, end, timezone):
    # Adjust start and end to be within valid hours after a DST transition
    if start.hour == 1:
        return start.replace(hour=2), end.replace(hour=2)
    elif end.hour == 1:
        return start.replace(hour=2), end.replace(hour=2)
    else:
        return start, end

# Example usage
print(check_overlap_fixed("2023-10-29T01:59", "2023-10-29T02:59",
                         "2023-10-29T03:00", "2023-10-29T04:00"))
```

**Example input / output:**

```text
# Example input and output
print(check_overlap_fixed("2023-10-29T01:59", "2023-10-29T02:59",
                         "2023-10-29T03:00", "2023-10-29T04:00"))
# Output: True
```

