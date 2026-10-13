---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would set up freshness monitoring that accounts for weekends and holidays when no data is expected.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distribution Change Alerts](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Freshness monitoring in data pipelines ensures that the data we consume or process has been updated within a reasonable time frame, typically measured from the last known update. However, setting up this monitoring can be more complex when dealing with weekends and holidays where no data is expected. Here’s how I would approach this problem, taking into account the needs of both intermediate and experienced engineers.

The core idea is to create a system that accurately tracks when data should have been updated based on business logic (e.g., excluding weekends and holidays) and sends alerts if these updates do not occur as expected. This involves defining the update schedule, handling exceptions for non-working days, and setting up automated notifications.

**Code:**

```python
# Example of a simple freshness monitoring function in Python

import pandas as pd
from datetime import timedelta

def get_expected_update_time(current_time):
    # Assuming standard business hours are 9 AM to 5 PM
    if current_time.hour < 9 or current_time.hour >= 17:
        return (current_time.replace(hour=16) + timedelta(days=1)).replace(minute=0, second=0)

    expected_update = current_time.replace(hour=16, minute=0, second=0)
    if pd.Timestamp(expected_update).weekday() > 4: # Friday
        expected_update += timedelta(days=3 - (pd.Timestamp(expected_update).weekday() % 5))

    return expected_update

# Example usage
from datetime import datetime

now = datetime.now()
print(f"Expected update time for {now}: {get_expected_update_time(now)}")
```

**Example input / output:**

```

Example input and output:

PLAINTEXT
Copy
Input: Current timestamp = 2023-10-09 14:00:00 (Wednesday)
Output: Expected update time = 2023-10-10 16:00:00

Input: Current timestamp = 2023-10-15 07:00:00 (Saturday)
Output: Expected update time = 2023-10-16 16:00:00
```

**Data:**

```text
No specific data schema is required here, but we can assume the system receives timestamps as part of the incoming data.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is it a working day?
7. Calculate next update time
8. Is today a holiday?
9. Set expected update to nextbusiness day
10. Return expected update time
11. End

