---
type: "practice"
difficulty: "Basic"
---

# How do you get the current date and time in Python and format it as a timestamp for a log message or a backup filename?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Comparing Time Windows](<../../../../TOOLS/Python/Python_Language/Comparing_Time_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To get the current date and time in Python, we can use the datetime module along with strftime to format it into a timestamp. This is particularly useful for logging messages or generating filenames that reflect the exact moment of an event.

#### Triaging the Question

The question asks how to retrieve the current date and time in Python and format it as a timestamp. The interviewer might be probing your understanding of common Python modules, string formatting methods, and best practices for handling timestamps. We will assume you are familiar with basic Python syntax and understand that strftime is used to convert dates into a readable string.

Step-by-Step Solution:
Import the Required Module:
The datetime module contains classes for manipulating dates and times.
Retrieve the Current Date and Time:
Use the datetime.now() method from the datetime module to get the current date and time.
Format the Timestamp:
Use the strftime method to format the datetime object into a string that resembles a timestamp.
Code Implementation:
PYTHON
from datetime import datetime

# Get the current date and time
current_datetime = datetime.now()

# Format it as a timestamp (YYYYMMDD_HHMMSS)
timestamp_format = "%Y%m%d_%H%M%S"
formatted_timestamp = current_datetime.strftime(timestamp_format)

print(f"Current Timestamp: {formatted_timestamp}")

#### Example
Output:
PLAINTEXT
Current Timestamp: 20231005_143045

#### Closing Summary
and Common Mistakes/Follow-Ups:

In this solution, we used the datetime module to get the current date and time. We formatted it using strftime with a specific format string that includes year (%Y), month (%m), day (%d), hour (%H), minute (%M), and second (%S). This approach ensures the timestamp is in a human-readable yet consistent format.

#### Common mistakes
include not importing the datetime module or using incorrect format strings, which could lead to errors. Follow-up questions might explore more complex formatting requirements or discuss how timestamps are used in logging versus backup filenames.

