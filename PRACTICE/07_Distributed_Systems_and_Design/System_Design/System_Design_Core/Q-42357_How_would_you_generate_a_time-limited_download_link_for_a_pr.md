---
type: "practice"
difficulty: "Intermediate"
---

# How would you generate a time-limited download link for a private file?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ URL Shortener](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/URL_Shortener.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To generate a time-limited download link for a private file, we need to ensure that the file is accessible only within a certain timeframe. This involves creating a unique token or URL that can be used by authorized users and will expire after a specific duration. The solution requires understanding how to securely manage access tokens, handle expiration logic, and ensure data integrity.

#### Triage

The interviewer is probing your knowledge of security best practices, token generation, URL shortening, and handling time-sensitive operations. We need to assume that the file is stored in a secure backend system (e.g., Amazon S3) and that we want to provide limited-time access through a generated link.

#### First-Principles Solution
Token Generation: Create a unique token for each download request.
Expiry Time: Define an expiry time for the token, ensuring it is short-lived (e.g., 5 minutes).
URL Construction: Construct a URL that includes the generated token and expiry time.
Backend Validation: Ensure the backend can validate the token before serving the file.
Step-by-Step Solution

Token Generation:

Generate a unique token using UUID or a secure random number generator.
Store the token in a database along with its expiration timestamp, associated user ID, and file metadata (e.g., S3 object key).

Expiry Time:

Set an expiry time for the token to ensure security and prevent unauthorized access.

URL Construction:

#### Cons
truct a URL that includes the generated token and expiry time.

#### Example
https://example.com/download?token=<TOKEN>&expiry=<EXPIRY_TIMESTAMP>

Backend Validation:

When a user requests to download the file, validate the token against the stored entry in the database.
Check if the current timestamp is within the expiry range and ensure that the associated user ID matches.

#### Closing Summary

Generating a time-limited download link involves creating a unique token for each request, storing it securely with its metadata, constructing an appropriate URL, and validating the token on the backend. This ensures that only authorized users can access the file and that the access is time-bound.

#### Common Mistakes/Follow-Ups
Common Mistake: Not properly securing the generation of tokens (e.g., using weak random number generators).

#### Follow-Up
Discussing additional security measures such as rate limiting, IP whitelisting, or multi-factor authentication.
Common Mistake: Expiring the token too soon, leading to user frustration; or expiring it too late, increasing security risks.

**Comparison:**

```
Token	Expiry Timestamp
9e3b5a8f-7b64-4c10-b2eb-d5f40c94e778	1633072800
```

**Code:**

```python
import uuid
from datetime import datetime, timedelta

def generate_token(expiry_duration=5):
    expiry_timestamp = int(datetime.now().timestamp()) + (expiry_duration * 60)
    token = str(uuid.uuid4())
    return {'token': token, 'expiry_timestamp': expiry_timestamp}

# Example usage:
token_info = generate_token()
print(token_info)
```

**Example input / output:**

```text
Input:
Generate a token with an expiry of 5 minutes.

Output:
{
  "token": "9e3b5a8f-7b64-4c10-b2eb-d5f40c94e778",
  "expiry_timestamp": 1633072800
}
```

**Data:**

```json
{
  "token": "9e3b5a8f-7b64-4c10-b2eb-d5f40c94e778",
  "expiry_timestamp": 1633072800
}
```

**Flow:**

1. Valid
2. Invalid
3. User Requests Download
4. Generate Token
5. Validate Token
6. Download File
7. Deny Access

