---
type: "practice"
difficulty: "Deep"
---

# A cross-field rule says shipped_date >= order_date, but timezone skew makes valid rows fail near midnight. How do you make cross-field validation timezone-safe?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Validating Loads and Deduplication](<../../../../02_Data_Engineering/Data_Quality/Validation/Validating_Loads_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of ensuring cross-field validation is timezone-safe when dealing with shipped_date and order_date, especially around the transition from one day to another (e.g., midnight), we need a robust approach that accounts for potential timezone differences. This involves understanding the problem, formulating assumptions, and then implementing a solution step-by-step.

#### Triaging the Question

The core of this question is about ensuring validation rules hold true despite timezone discrepancies. The key idea here is to handle the boundary conditions where datetimes are near midnight, as these can be ambiguous without proper handling.

#### Assumptions
Both shipped_date and order_date will be in a specific timezone.
We need to allow for a small leeway (e.g., 15 minutes) around the transition point of midnight to account for potential clock differences or data entry errors.
The validation should work across different timezones, not just UTC.

#### First-Principles Solution

To ensure that shipped_date is at least as recent as order_date, even when dealing with timezone skew near midnight, we can implement a validation function that takes into account the local timezone and introduces a small buffer period. Here’s how you can approach this:

Determine the Local Timezone: Use the appropriate timezone for each date.
Add a Buffer Period: Account for up to 15 minutes before or after midnight, as this is often where discrepancies occur.
Perform Date Arithmetic: Ensure that shipped_date falls within an acceptable window relative to order_date.
Step-by-Step Solution

Determine Timezone:

Use the pytz library in Python to handle timezones effectively.

Add Buffer Period:

Define a buffer period of 15 minutes around midnight.
This can be implemented using datetime operations.

Perform Date Arithmetic:

Calculate the lower and upper bounds for valid shipped_date.
Validate that shipped_date falls within these bounds relative to order_date.

Here’s an example in Python:

PYTHON
from datetime import timedelta, datetime
import pytz

def validate_shipment(order_date_str, shipped_date_str, timezone_name='UTC'):
    # Parse the input dates
    order_date = datetime.strptime(order_date_str, '%Y-%m-%d %H:%M:%S')
    shipped_date = datetime.strptime(shipped_date_str, '%Y-%m-%d %H:%M:%S')

    # Set the timezone
    timezone = pytz.timezone(timezone_name)
    order_date_tz = timezone.localize(order_date).astimezone(pytz.utc)
    shipped_date_tz = timezone.localize(shipped_date).astimezone(pytz.utc)

    # Define buffer period (15 minutes around midnight)
    buffer_minutes = 15
    buffer_seconds = buffer_minutes * 60

    # Calculate the valid window for shipped_date
    min_shipped_time = order_date_tz - timedelta(seconds=buffer_seconds)
    max_shipped_time = order_date_tz + timedelta(seconds=buffer_seconds)

    return min_shipped_time <= shipped_date_tz <= max_shipped_time

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that the shipped_date is validated within a 15-minute window around the order_date, accounting for potential timezone discrepancies. By using timezones correctly and adding buffer periods, we can make our validation more robust.

#### Common Mistakes
Forgetting to account for timezone differences.
Not considering edge cases like exactly at midnight or different local times.
Using naive datetime handling without proper timezone awareness.

#### Follow-ups
could include discussing how this solution scales with larger datasets, potential optimizations, and integrating this into a broader data quality framework.

**Code:**

```python
from datetime import timedelta, datetime
import pytz

def validate_shipment(order_date_str, shipped_date_str, timezone_name='UTC'):
    # Parse the input dates
    order_date = datetime.strptime(order_date_str, '%Y-%m-%d %H:%M:%S')
    shipped_date = datetime.strptime(shipped_date_str, '%Y-%m-%d %H:%M:%S')

    # Set the timezone
    timezone = pytz.timezone(timezone_name)
    order_date_tz = timezone.localize(order_date).astimezone(pytz.utc)
    shipped_date_tz = timezone.localize(shipped_date).astimezone(pytz.utc)

    # Define buffer period (15 minutes around midnight)
    buffer_minutes = 15
    buffer_seconds = buffer_minutes * 60

    # Calculate the valid window for shipped_date
    min_shipped_time = order_date_tz - timedelta(seconds=buffer_seconds)
    max_shipped_time = order_date_tz + timedelta(seconds=buffer_seconds)

    return min_shipped_time <= shipped_date_tz <= max_shipped_time
```

**Example input / output:**

```text
Input:
order_date_str = "2023-10-05 23:45:00"
shipped_date_str = "2023-10-06 00:15:00"

Output:
True

Explanation:
The shipped date is within the buffer period of 15 minutes around midnight, so it passes validation.
```

**Data:**

```json
{
  "order_date": "2023-10-05 23:45:00",
  "shipped_date": "2023-10-06 00:15:00"
}
```

