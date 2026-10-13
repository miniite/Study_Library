---
type: "practice"
difficulty: "Basic"
---

# How can you use a regular expression to validate that an email column has a proper format?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validating Loads and Deduplication](<../../../../02_Data_Engineering/Data_Quality/Validation/Validating_Loads_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To validate whether an email column contains properly formatted emails, we can leverage regular expressions (regex). Regex allows us to specify patterns that match certain structures, making it ideal for validating email formats. In this context, a basic regex pattern can ensure the email follows the common structure of username@domain.com.

#### Triaging the Question

The question is essentially asking how to use regex to validate an email column. The interviewer might be probing your understanding of:

Basic syntax and usage of regular expressions.
Common patterns for validating email formats.
How to apply this validation in a real-world scenario.

#### First Principles Solution

Understand the Regex Pattern: A typical regex pattern for email validation is ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$. This pattern ensures:

The username part contains alphanumeric characters (including ., _, %, +, -).
There's an @ symbol.
The domain name consists of alphanumeric characters and possibly periods (.).
A top-level domain (TLD) is present, with at least two letters.

#### Implementation
in Python: We can use Python’s re module to apply this regex pattern to the email column.

Handling Edge Cases: Consider edge cases like empty strings, leading/trailing spaces, and invalid characters.

Integration into Data Pipeline: Ensure that validation is part of your data cleaning or ETL pipeline.

Full Solution

Here's how you can implement a basic email validation in Python:

PYTHON
import re

def validate_email(email):
    # Define the regex pattern for an email address
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'

    # Use re.match to check if the email matches the pattern
    match = re.match(pattern, email)

    return bool(match)

# Example usage:
emails = ["test@example.com", "invalid-email@", "@example.com", "goodemail@domain.com"]
valid_emails = [email for email in emails if validate_email(email)]
print("Valid Emails:", valid_emails)

#### Closing Summary
and Common Mistakes

This solution uses a regex pattern to validate email formats. The most common mistakes include:

Overlooking edge cases like leading/trailing spaces or invalid characters.
Not handling non-standard TLDs (e.g., .io, .biz).
Failing to account for special characters allowed in usernames (., _, %, +, -).

Always consider the specific requirements of your project when defining the regex pattern. For instance, if you need to validate more complex or non-standard email formats, a different pattern may be necessary.

Code Blocks
PYTHON
import re

def validate_email(email):
    # Define the regex pattern for an email address
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'

    # Use re.match to check if the email matches the pattern
    match = re.match(pattern, email)

    return bool(match)

# Example usage:
emails = ["test@example.com", "invalid-email@", "@example.com", "goodemail@domain.com"]
valid_emails = [email for email in emails if validate_email(email)]
print("Valid Emails:", valid_emails)
Self-Check

Correctness + completeness confirmed.

