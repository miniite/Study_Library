---
type: "practice"
difficulty: "Deep"
---

# You must verify inbound signed webhooks from a payment provider in Python. How do you validate the HMAC signature in constant time, reject replays with a timestamp window, and handle key rotation?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Global Quotas Across Requests](<../../../../TOOLS/Python/Python_Language/Global_Quotas_Across_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the integrity of inbound signed webhooks from a payment provider using Python, we need to implement three main checks: validating the HMAC signature in constant time, rejecting replay attacks with a timestamp window, and handling key rotation. This involves leveraging libraries like hmac for signing verification, setting up an effective timestamp validation mechanism, and implementing a strategy for key management.

**Comparison:**

| Key          | Value                     |
|--------------|---------------------------|
| Secret Keys  | SECRET_KEY1, SECRET_KEY2   |
| Timestamp    | 1633072800 (2021-09-23)   |
| Signature    | 9e3b4a8d3e2f7c2f0d6e5c7b1f4d0a1e |

**Code:**

```python
import hmac
from hashlib import sha256
from time import time
from typing import Optional

def validate_webhook_signature(secret_key: str, payload: bytes, received_signature: str) -> bool:
    """
    Validate the HMAC signature of a webhook in constant time.
    """
    expected_signature = hmac.new(
        key=secret_key.encode('utf-8'),
        msg=payload,
        digestmod=sha256
    ).hexdigest()

    return hmac.compare_digest(expected_signature, received_signature)

def validate_timestamp(received_timestamp: Optional[str], timestamp_window_seconds: int) -> bool:
    """
    Validate the timestamp to ensure it's within a given window.
    """
    current_time = time()
    received_timestamp_float = float(received_timestamp)

    if not received_timestamp or abs(current_time - received_timestamp_float) > timestamp_window_seconds:
        return False

    return True

def handle_key_rotation(secret_keys: list[str], payload: bytes, received_signature: str) -> bool:
    """
    Handle key rotation by checking against multiple secret keys.
    """
    for key in secret_keys:
        if validate_webhook_signature(key, payload, received_signature):
            return True
    return False

def main():
    # Example usage
    secret_keys = ['SECRET_KEY1', 'SECRET_KEY2']
    payload = b'{"amount": 50.0, "currency": "USD"}'
    received_timestamp = '1633072800'  # Unix timestamp in seconds
    received_signature = '9e3b4a8d3e2f7c2f0d6e5c7b1f4d0a1e'

    if validate_webhook_signature(secret_keys[0], payload, received_signature):
        print("Signature valid.")
    elif handle_key_rotation(secret_keys, payload, received_signature):
        print("Signature valid with key rotation.")
    else:
        print("Invalid signature or replay attack detected.")

if __name__ == "__main__":
    main()
```

**Example input / output:**

| Input | Expected Output |
|-------|----------------|
| secret_keys = ['SECRET_KEY1', 'SECRET_KEY2']<br>payload = b'{"amount": 50.0, "currency": "USD"}'<br>received_timestamp = '1633072800'<br>received_signature = '9e3b4a8d3e2f7c2f0d6e5c7b1f4d0a1e' | Signature valid. |

**Data:**

```yaml
secret_keys: ['SECRET_KEY1', 'SECRET_KEY2']
payload: '{"amount": 50.0, "currency": "USD"}'
received_timestamp: '1633072800' # Unix timestamp in seconds
received_signature: '9e3b4a8d3e2f7c2f0d6e5c7b1f4d0a1e'
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Receive Webhook
8. Parse Payload
9. Extract Timestamp
10. Validate Timestamp
11. Calculate Signature
12. Compare Received Signature
13. Signature Valid?
14. Process Webhook
15. Reject Request
16. Late or Invalid Timestamp?
17. Reject Request
18. Check Key Rotation
19. Key Found?
20. Process Webhook
21. Reject Request

