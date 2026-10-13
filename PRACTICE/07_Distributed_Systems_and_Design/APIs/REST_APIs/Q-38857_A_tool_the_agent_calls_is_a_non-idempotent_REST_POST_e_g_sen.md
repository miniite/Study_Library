---
type: "practice"
difficulty: "Deep"
---

# A tool the agent calls is a non-idempotent REST POST (e.g. send email). The model sometimes retries or duplicates the call. How do you make tool execution safe against duplicate model-initiated calls within one turn?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent POST Requests](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Idempotent_POST_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of making non-idempotent REST POST calls (like sending an email) safe from being retried or duplicated by a model, we need to ensure that such calls are idempotent and unique within one turn. This can be achieved through several mechanisms, including using retries with exponential backoff, ensuring atomicity in the call process, and leveraging transactional state management.

#### Triaging the Question

The key challenge here is handling non-idempotent operations, which typically require a response to confirm success or failure. Retrying such calls without proper identification can lead to duplicate emails being sent. The problem also asks for making these calls safe within one turn, implying that we need to manage state across multiple retries effectively.

Step-by-Step Solution
Identify the Unique Request: Each non-idempotent call should be uniquely identifiable.
Retries with Backoff: Implement exponential backoff to manage retries gracefully.
Atomic Transaction Management: Use atomic transactions to ensure that the call is either fully executed or rolled back if it fails partially.
Step 1: Identify the Unique Request

We can use a combination of metadata and timestamps to make each request unique. For instance, we could include an x-request-id header in our HTTP requests. Additionally, tracking the timestamp or sequence number of the call helps in identifying and managing retries.

PYTHON
def generate_request_id():
    return str(uuid.uuid4())
Step 2: Retries with Backoff

Implementing exponential backoff is crucial to avoid overwhelming the system with too many retries immediately after a failure. This involves waiting for an increasing amount of time between each retry attempt.

PYTHON
def send_email(email_data):
    request_id = generate_request_id()

    while True:
        try:
            response = requests.post("https://api.example.com/send-email", json=email_data, headers={"x-request-id": request_id})
            if response.status_code == 200:
                return True
            else:
                # Handle non-200 responses appropriately (e.g., log and retry)
                break
        except Exception as e:
            # Log the error for debugging purposes
            logging.error(f"Failed to send email {email_data['to']} with request id {request_id}: {str(e)}")

        # Exponential backoff: wait for 2^attempt * base_delay seconds before retrying
        time.sleep(2 ** (attempt - 1) * BASE_DELAY)
Step 3: Atomic Transaction Management

To ensure atomicity, we can use transactional state management to keep track of which emails have been sent. This can be done using a database or even in-memory storage if the scale is manageable.

PYTHON
class EmailSender:
    def __init__(self):
        self.sent_emails = set()

    def send_email(self, email_data):
        request_id = generate_request_id()

        while True:
            try:
                response = requests.post("https://api.example.com/send-email", json=email_data, headers={"x-request-id": request_id})
                if response.status_code == 200 and request_id not in self.sent_emails:
                    self.sent_emails.add(request_id)
                    return True
                else:
                    # Handle non-200 responses appropriately (e.g., log and retry)
                    break
            except Exception as e:
                # Log the error for debugging purposes
                logging.error(f"Failed to send email {email_data['to']} with request id {request_id}: {str(e)}")

            # Exponential backoff: wait for 2^attempt * base_delay seconds before retrying
            time.sleep(2 ** (attempt - 1) * BASE_DELAY)

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing these strategies, we ensure that each email sending call is unique and idempotent within one turn. This approach prevents duplicate emails being sent unnecessarily, maintaining the integrity of our system.

#### Common Mistakes

Failing to implement exponential backoff correctly can lead to a high volume of retries in a short time.
Not tracking which requests have been successfully completed can result in re-sending duplicates.
Ignoring non-200 responses without proper handling can cause state inconsistencies.
Trialing the Solution

To validate this solution, we could set up a mock environment with multiple parallel calls to simulate retries and check that duplicate emails are not sent. We would also need to ensure logging is in place for debugging purposes.

PYTHON
def test_email_sender():
    sender = EmailSender()

    email_data1 = {"to": "user1@example.com", "subject": "Test 1"}
    sender.send_email(email_data1)

    time.sleep(20)  # Simulate a delay to trigger exponential backoff

    email_data2 = {"to": "user2@example.com", "subject": "Test 2"}
    sender.send_email(email_data2)

**Comparison:**

| Attempt | Wait Time (seconds) |
|---------|---------------------|
| 1       | 1                   |
| 2       | 2                   |
| 3       | 4                   |
| 4       | 8                   |

**Code:**

```python
import requests
from uuid import uuid4
import time

def generate_request_id():
    return str(uuid.uuid4())

class EmailSender:
    def __init__(self):
        self.sent_emails = set()

    def send_email(self, email_data):
        request_id = generate_request_id()

        while True:
            try:
                response = requests.post("https://api.example.com/send-email", json=email_data, headers={"x-request-id": request_id})
                if response.status_code == 200 and request_id not in self.sent_emails:
                    self.sent_emails.add(request_id)
                    return True
                else:
                    # Handle non-200 responses appropriately (e.g., log and retry)
                    break
            except Exception as e:
                # Log the error for debugging purposes
                logging.error(f"Failed to send email {email_data['to']} with request id {request_id}: {str(e)}")

            # Exponential backoff: wait for 2^attempt * base_delay seconds before retrying
            time.sleep(2 ** (attempt - 1) * BASE_DELAY)
```

**Example input / output:**

```text
sender = EmailSender()
sent_email_status = sender.send_email(email_data1)
assert sent_email_status, "Failed to send first email"

time.sleep(20)  # Simulate a delay

sent_email_status = sender.send_email(email_data2)
assert sent_email_status, "Failed to send second email"
```

**Data:**

```json
{
    "email_data1": {"to": "user1@example.com", "subject": "Test 1"},
    "email_data2": {"to": "user2@example.com", "subject": "Test 2"}
}
```

**Flow:**

1. 200
2. !200
3. Generate Request ID
4. Send Email
5. Response Status
6. Mark as Sent, Add to Set
7. Handle Error, Retry withBackoff

