---
type: "practice"
difficulty: "Deep"
---

# The assistant wrote an at-least-once consumer but the downstream side effect is a non-idempotent email send. How would you catch the duplicate-email risk and prompt for a dedupe-before-send design?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prompting for Production Patterns](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Production_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The key issue here is ensuring that emails are not sent multiple times, which could be problematic if an email send operation is non-idempotent. We need to catch duplicate risk by implementing a deduplication mechanism before the email is sent. This can be achieved through a combination of state tracking and retry logic.

#### Triaging the Question

The core question revolves around handling non-idempotent operations (like sending emails) in an at-least-once delivery system, ensuring that each email is only sent once to avoid duplication issues. The interviewer is probing your understanding of idempotence, state management, and robust error handling strategies.

#### Assumptions
Non-Idempotent Operation: Email send operation can fail and may need to be retried.
At-Least-Once Delivery Guarantee: The system guarantees that the message will be delivered at least once, but not necessarily only once.

#### First Principles Solution
Step 1: Identifying the Issue

Non-idempotent operations like email sends can lead to duplicate emails if handled naively in an at-least-once delivery setup. To catch and mitigate this risk, we need a deduplication strategy that ensures each unique message (email) is processed exactly once.

Step 2: Designing the Deduplication Mechanism
Message Identification: Each email must be uniquely identified.
State Tracking: Maintain an in-memory or database state to track which emails have already been sent.
Check Before Send: Before sending an email, check if it has already been processed.
Step 3: Implementation Details
Option 1: In-Memory State

#### Pros
Fast lookups and no external dependencies.

#### Cons
Not durable; state is lost on restarts or crashes.
PYTHON
class EmailSender:
    def __init__(self, email_id_extractor):
        self.email_ids_sent = set()

    def send_email(self, message):
        email_id = email_id_extractor(message)
        if email_id in self.email_ids_sent:
            return f"Duplicate: {email_id}"
        else:
            # Send the email
            print(f"Sending {message}")
            self.email_ids_sent.add(email_id)
Option 2: Database State Tracking

#### Pros
Persistent state that can survive restarts.

#### Cons
Slightly slower lookups.
PYTHON
import sqlite3

class EmailSender:
    def __init__(self, email_id_extractor):
        self.conn = sqlite3.connect('email_tracker.db')
        cursor = self.conn.cursor()
        cursor.execute('''CREATE TABLE IF NOT EXISTS sent_emails (id TEXT PRIMARY KEY)''')
        self.conn.commit()

    def send_email(self, message):
        email_id = email_id_extractor(message)
        with self.conn:
            if self.conn.execute("SELECT 1 FROM sent_emails WHERE id=?", (email_id,)).fetchone():
                return f"Duplicate: {email_id}"
            else:
                # Send the email
                print(f"Sending {message}")
                cursor = self.conn.cursor()
                cursor.execute("INSERT INTO sent_emails VALUES (?)", (email_id,))
Step 4: Handling Retries and Deduplication
Idempotent Retry: If an email send fails, retry sending it. However, ensure that the same operation is not retried if it has already been processed.
PYTHON
def send_email_with_retry(email_sender, message):
    for _ in range(3):  # Example: 3 retries
        result = email_sender.send_email(message)
        if "Duplicate" not in result:
            break
Step 5: Integration and Testing
Ensure the deduplication mechanism is robust by testing edge cases like:
Sending duplicate emails.
Handling retry failures.
Managing state consistency across restarts.

#### Closing Summary
and Common Mistakes:

To avoid sending duplicate emails, it’s essential to implement a deduplication strategy that checks for processed emails before attempting to send. This can be achieved using in-memory or database states, along with proper error handling and retries. A common mistake is not properly tracking the state of sent emails, leading to potential duplication issues.

#### Follow-Up
Questions:
How would you scale this solution for a large number of unique messages?
What if the email send operation has side effects (like updating user profiles)?
How do you handle concurrent access in your deduplication mechanism?

**Comparison:**

| Email | Sent |
|-------|------|
| A     | Yes  |
| B     | No   |
| C     | Yes  |

**Code:**

```python
class EmailSender:
    def __init__(self, email_id_extractor):
        self.email_ids_sent = set()

    def send_email(self, message):
        email_id = email_id_extractor(message)
        if email_id in self.email_ids_sent:
            return f"Duplicate: {email_id}"
        else:
            # Send the email
            print(f"Sending {message}")
            self.email_ids_sent.add(email_id)

class EmailSenderDB:
    def __init__(self, email_id_extractor):
        self.conn = sqlite3.connect('email_tracker.db')
        cursor = self.conn.cursor()
        cursor.execute('''CREATE TABLE IF NOT EXISTS sent_emails (id TEXT PRIMARY KEY)''')
        self.conn.commit()

    def send_email(self, message):
        email_id = email_id_extractor(message)
        with self.conn:
            if self.conn.execute("SELECT 1 FROM sent_emails WHERE id=?", (email_id,)).fetchone():
                return f"Duplicate: {email_id}"
            else:
                # Send the email
                print(f"Sending {message}")
                cursor = self.conn.cursor()
                cursor.execute("INSERT INTO sent_emails VALUES (?)", (email_id,))
```

**Example input / output:**

```text
# Example of sending an email
sender = EmailSenderDB(lambda x: x['subject'])
result = sender.send_email({'subject': 'Welcome to our platform', 'body': 'Thank you for joining!'})
print(result)  # Output: Sending Welcome to our platform

# Attempting to send the same email again
result = sender.send_email({'subject': 'Welcome to our platform', 'body': 'Thank you for joining!'})
print(result)  # Output: Duplicate: Welcome to our platform
```

**Data:**

```sql
CREATE TABLE IF NOT EXISTS sent_emails (
    id TEXT PRIMARY KEY
);
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Receive Message] --> B{Is Email Id in State?};
4. B -- Yes --> C{Send Email};
5. C -- Success --> D[Return "Sent"];
6. B -- No --> E[Store Email Id];
7. E --> F{Send Email};
8. F -- Success --> G[Return "Sent"];
9. F -- Failed --> H[Retry or Log Failure]

