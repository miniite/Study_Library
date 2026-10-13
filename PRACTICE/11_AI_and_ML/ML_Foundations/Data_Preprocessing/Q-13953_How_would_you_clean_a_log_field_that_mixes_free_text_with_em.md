---
type: "practice"
difficulty: "Intermediate"
---

# How would you clean a log field that mixes free text with embedded numbers, like an error message containing a user ID and a duration?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Parsing Semi-structured Logs](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Parsing_Semi-structured_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To clean a log field that mixes free text with embedded numbers, such as an error message containing a user ID and a duration, you need to extract structured data from unstructured text. This involves identifying patterns (like IDs or durations) within the text and isolating them for further analysis. The key steps are: identifying patterns, extracting relevant information, and formatting the extracted data.

#### Triaging the Question

The question is asking how to process a log message that contains mixed content, specifically identifying and extracting user IDs and durations from free text. The interviewer is likely probing your understanding of regular expressions (regex), string manipulation techniques, and data cleaning best practices. We need to make assumptions such as knowing the typical format of the log entries or having examples to work with.

#### First-Principles Solution
Identify Patterns: Use regex to identify patterns like user IDs and durations.
Extract Information: Apply regex to extract these patterns from the text.
Format Output: Format the extracted data into a structured format suitable for further processing or storage.
Step-by-Step Solution

Identifying Patterns:

User ID: Typically alphanumeric, e.g., user_007.
Duration: A number with units, e.g., 5min.

Extracting Information:

PYTHON
import re

# Sample log message
log_message = "User user_007 experienced an error lasting 3days"

# Regex to find a typical alphanumeric user ID
user_id_pattern = r"(\buser_\d{3}\b)"
# Regex to find a duration in days, hours, or minutes
duration_pattern = r"(\b\d+day|hour|minute\b)"

# Extract user ID and duration using regex
user_id_match = re.search(user_id_pattern, log_message)
if user_id_match:
    user_id = user_id_match.group(0)
    user_id = user_id.replace("user_", "")  # Clean the ID

duration_match = re.findall(duration_pattern, log_message)
durations = [d for d in duration_match if 'day' in d]
if durations:
    total_duration_in_days = int(durations[0].split('day')[0])
    print(f"User ID: {user_id}, Duration in Days: {total_duration_in_days}")
else:
    print("No valid duration found.")

Formatting Output:

The output should be in a structured format, such as a dictionary or JSON object.
PYTHON
result = {
    "user_id": user_id,
    "duration_days": total_duration_in_days
}
print(result)

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that the log message is cleaned by extracting meaningful data. However, there are a few common mistakes:

Incomplete Data Handling: Not handling cases where no user ID or duration exists.
Regex Overcomplication: Using overly complex regex patterns which can be hard to maintain.

For follow-ups, you might discuss how to handle more complex log formats, validate the extracted data, and optimize for performance with larger datasets.

