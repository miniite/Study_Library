---
type: "practice"
difficulty: "Intermediate"
---

# How would you use Python to compute the total number of requests per minute from a list of request timestamps for a traffic dashboard?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Comparing Time Windows](<../../../../TOOLS/Python/Python_Language/Comparing_Time_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to count the number of requests that fall within each minute interval from a list of timestamp strings. This involves parsing the timestamps and categorizing them by minutes. Here’s how you can approach it step-by-step:

Understand the Requirement: The task is to compute the total number of HTTP requests per minute from a list of request timestamps.

#### Assumptions
Timestamps are in ISO 8601 format (YYYY-MM-DDTHH:MM:SS).
We need to handle potential edge cases such as timestamps that span multiple minutes.

#### Approach
Parse each timestamp to get the minute part.
Use a dictionary to keep track of request counts per minute.

Now, let's break down the solution from first principles:

Step-by-Step Solution
Parse Timestamps: Convert the timestamp strings into datetime objects and extract the minute component.
Categorize by Minute: Use a dictionary where the keys are minutes (as strings or integers) and values are counts of requests for that minute.
Count Requests: Iterate through the list, categorize each request to its corresponding minute, and update the count in the dictionary.
Code Implementation
PYTHON
from datetime import datetime

def compute_requests_per_minute(timestamps):
    # Initialize a dictionary to store the count per minute
    requests_per_minute = {}

    for timestamp_str in timestamps:
        try:
            # Parse the timestamp string
            dt = datetime.fromisoformat(timestamp_str)

            # Extract the minute component
            minute = dt.strftime('%Y-%m-%d %H:%M')

            # Increment the count for this minute
            if minute not in requests_per_minute:
                requests_per_minute[minute] = 0
            requests_per_minute[minute] += 1
        except ValueError as e:
            print(f"Invalid timestamp: {timestamp_str}. Error: {e}")

    return requests_per_minute

# Example timestamps
timestamps = [
    "2023-10-05T14:15:00",
    "2023-10-05T14:16:00",
    "2023-10-05T15:17:00",
    "2023-10-05T15:18:00"
]

result = compute_requests_per_minute(timestamps)
print(result)
Explanation
Timestamp Parsing: We use datetime.fromisoformat to parse the timestamp strings into datetime objects.
Minute Extraction: We format each parsed datetime object back to a string in the 'YYYY-MM-DD HH:MM' format to extract the minute component.
Counting Requests: We increment the count for each unique minute. If a new minute appears, it is added to the dictionary.

#### Common Mistakes
and Follow-Ups
Incorrect Parsing: Ensure that timestamps are correctly formatted. Incorrect formatting can lead to ValueError.
Handling Multiple Time Zones: If the timestamps include time zones, ensure they are correctly parsed.
Edge Cases: Consider cases where a timestamp might span multiple minutes (e.g., "2023-10-05T14:59:00" to "2023-10-05T15:01:00").

By following these steps and handling potential edge cases, you can ensure that the solution works robustly.

