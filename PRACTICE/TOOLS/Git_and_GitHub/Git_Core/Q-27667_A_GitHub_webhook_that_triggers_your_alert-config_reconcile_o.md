---
type: "practice"
difficulty: "Deep"
---

# A GitHub webhook that triggers your alert-config reconcile occasionally fires twice for one push, causing two overlapping rule-reload deploys and a brief double-page. How do you make the consumer idempotent and validate the webhook signature so a replay can't re-trigger a deploy?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Secure Workflows for Forks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Secure_Workflows_for_Forks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that our GitHub webhook consumer is idempotent—meaning it should handle duplicate or replayed requests in the same way as an initial request without causing any unintended side effects. Additionally, we must validate the webhook signature to prevent replay attacks and ensure that only legitimate triggers are processed. This involves both technical implementation details and strategic design choices.

The question probes your understanding of webhook handling in a distributed system, idempotency principles, and security best practices like signature validation. We'll break down the solution by first defining what an idempotent consumer is, then implementing it, and finally ensuring the webhook's signature can be validated securely.

**Code:**

```python
import hmac
import json
from hashlib import sha256
from urllib.parse import parse_qs

# Configuration for webhook secret key
SECRET_KEY = 'your-secret-key'

def is_idempotent(action):
    """Check if the action has already been processed."""
    # This could be stored in a database or file system
    actions_processed = set()
    return action not in actions_processed

def validate_webhook_signature(request_body, query_string, secret_key):
    """Validate the webhook signature for replay protection."""
    received_signature_header = request.headers.get('X-Hub-Signature')
    if not received_signature_header:
        raise ValueError("Missing X-Hub-Signature header")

    sha_name, signature = received_signature_header.split('=')
    if sha_name != 'sha256':
        raise ValueError(f"Unsupported hash algorithm: {sha_name}")

    expected_signature = hmac.new(
        secret_key.encode('utf-8'),
        msg=request_body.encode('utf-8') + parse_qs(query_string).encode('utf-8'),
        digestmod=sha256
    ).hexdigest()

    return signature == expected_signature

def handle_webhook_event():
    """Handle the incoming webhook event."""
    # Parse request body and query string
    body = request.data
    query_string = parse_qs(request.query_string.decode('utf-8'))

    if not validate_webhook_signature(body, query_string['hub.challenge'][0], SECRET_KEY):
        raise ValueError("Invalid signature")

    action = json.loads(body)

    # Ensure the action is idempotent
    if is_idempotent(action):
        # Process the action (e.g., reload deploy configurations)
        print(f"Processing {action}")
    else:
        print(f"Ignoring duplicate request for {action}")

handle_webhook_event()
```

**Example input / output:**

```text
Request Body (JSON):
{
  "ref": "refs/heads/main",
  "before": "a1b2c3d4e5f6",
  "after": "1234abcd"
}

Query String:
hub.challenge=unique_challenge_token

Response:
Processing { "ref": "refs/heads/main", "before": "a1b2c3d4e5f6", "after": "1234abcd" }
```

**Flow:**

1. Yes
2. No
3. Receive Webhook Request
4. Validate Signature
5. Is Idempotent?
6. Process Action
7. Ignore Duplicate Request

