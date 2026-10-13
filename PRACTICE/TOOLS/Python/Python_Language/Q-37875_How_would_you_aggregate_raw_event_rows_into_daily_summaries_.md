---
type: "practice"
difficulty: "Basic"
---

# How would you aggregate raw event rows into daily summaries in Python, and how does grouping by a date key produce a metrics table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Data Basics](<../../../../TOOLS/Python/Python_Language/Time-series_Data_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To aggregate raw event rows into daily summaries using Python, we'll focus on processing each row based on its timestamp to group them by day. This involves parsing dates, summarizing events, and storing or returning aggregated metrics in a structured format. We can achieve this using pandas, a powerful library for data manipulation in Python.

**Code:**

```python
import pandas as pd

# Sample input data: A list of dictionaries representing event rows.
events = [
    {"timestamp": "2023-10-05 14:30:00", "user_id": 1, "event_type": "login"},
    {"timestamp": "2023-10-06 12:20:00", "user_id": 1, "event_type": "logout"},
    {"timestamp": "2023-10-05 19:45:00", "user_id": 2, "event_type": "login"},
    {"timestamp": "2023-10-06 08:00:00", "user_id": 3, "event_type": "login"}
]

# Convert the list of dictionaries to a DataFrame for easier manipulation.
df = pd.DataFrame(events)

# Extract the date part from the timestamp column and set it as the index.
df['timestamp'] = pd.to_datetime(df['timestamp'])
df.set_index('timestamp', inplace=True)
```

**Data:**

```json
[
    {"timestamp": "2023-10-05 14:30:00", "user_id": 1, "event_type": "login"},
    {"timestamp": "2023-10-06 12:20:00", "user_id": 1, "event_type": "logout"},
    {"timestamp": "2023-10-05 19:45:00", "user_id": 2, "event_type": "login"},
    {"timestamp": "2023-10-06 08:00:00", "user_id": 3, "event_type": "login"}
]
```

