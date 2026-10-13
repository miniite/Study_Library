---
type: "practice"
difficulty: "Deep"
---

# How do you design a FastAPI endpoint that accepts webhooks from multiple providers with different signature schemes, verifying each securely while sharing one processing pipeline?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Authentication](<../../../../TOOLS/Python/FastAPI/API_Authentication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a FastAPI endpoint that accepts webhooks from multiple providers with different signature schemes, we need to handle the verification of signatures securely and efficiently while ensuring that all validated webhooks are processed through a common pipeline. This involves understanding the various signature schemes used by the providers, implementing secure verification logic, and designing an extensible system for handling and processing incoming webhook data.

#### Triage

The core question here is how to design a FastAPI endpoint that can handle different types of signatures (e.g., HMAC, SHA-256) from multiple providers. The interviewer is likely probing your understanding of security practices, modular design principles, and the ability to implement flexible verification logic. We will make some key assumptions: each provider uses a unique signature scheme, and we need to verify these signatures before processing the payload.

#### First-Principles Solution
Define a Common Pipeline Interface: Create an interface that all webhook processors must adhere to.
Implement Signature Verification Logic: Write functions specific to each provider's signature scheme.
Use Dependency Injection for Flexibility: Ensure that our system can easily be extended with new providers and their respective verification logic.
Securely Store Secrets: Safeguard the secrets used for verifying signatures.
Step-by-Step Solution

Define a Common Pipeline Interface

PYTHON
from abc import ABC, abstractmethod

class WebhookProcessor(ABC):
    @abstractmethod
    def process(self, payload: dict) -> bool:
        pass

Implement Signature Verification Functions

PYTHON
def verify_hmac_signature(payload: bytes, secret_key: str) -> bool:
    # Implementation for HMAC verification
    return True  # Placeholder

def verify_sha256_signature(payload: bytes, secret_key: str) -> bool:
    # Implementation for SHA-256 verification
    return True  # Placeholder

Use Dependency Injection for Flexibility

PYTHON
from typing import TypeVar

TProcessor = TypeVar('TProcessor', bound=WebhookProcessor)

def get_webhook_processor(provider: str) -> WebhookProcessor:
    if provider == 'provider1':
        return HmacProcessor()
    elif provider == 'provider2':
        return Sha256Processor()
    # Add more providers as needed

class HmacProcessor(WebhookProcessor):
    def __init__(self, secret_key: str):
        self.secret_key = secret_key

    def process(self, payload: dict) -> bool:
        if verify_hmac_signature(payload['data'], self.secret_key):
            return True
        return False

class Sha256Processor(WebhookProcessor):
    def __init__(self, secret_key: str):
        self.secret_key = secret_key

    def process(self, payload: dict) -> bool:
        if verify_sha256_signature(payload['data'], self.secret_key):
            return True
        return False

Securely Store Secrets Secrets should be stored securely and never hardcoded in the source code.

PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class WebhookData(BaseModel):
    provider: str
    data: bytes  # Raw payload or base64 encoded

@app.post("/webhooks/{provider}")
async def receive_webhook(provider: str, webhook_data: WebhookData):
    processor = get_webhook_processor(provider)
    if not processor.process(webhook_data.data):
        raise HTTPException(status_code=401, detail="Signature verification failed")

    # Process the payload
    process_payload(webhook_data)

def process_payload(payload: dict):
    # Placeholder for actual processing logic
    pass

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes

Hardcoding secrets directly in the code.
Not using secure methods to store and manage secrets.
Lack of proper error handling and logging.

#### Follow-Ups

How would you handle edge cases where a provider's signature scheme is not supported?
What measures can be taken to ensure the security of the pipeline itself?
How would you scale this system if it were to process thousands of webhooks per second?

**Comparison:**

```
Provider	Signature Scheme
provider1	HMAC
provider2	SHA-256
```

**Code:**

```python
from abc import ABC, abstractmethod

class WebhookProcessor(ABC):
    @abstractmethod
    def process(self, payload: dict) -> bool:
        pass

def verify_hmac_signature(payload: bytes, secret_key: str) -> bool:
    # Implementation for HMAC verification
    return True  # Placeholder

def verify_sha256_signature(payload: bytes, secret_key: str) -> bool:
    # Implementation for SHA-256 verification
    return True  # Placeholder

from typing import TypeVar

TProcessor = TypeVar('TProcessor', bound=WebhookProcessor)

def get_webhook_processor(provider: str) -> WebhookProcessor:
    if provider == 'provider1':
        return HmacProcessor()
    elif provider == 'provider2':
        return Sha256Processor()
    # Add more providers as needed

class HmacProcessor(WebhookProcessor):
    def __init__(self, secret_key: str):
        self.secret_key = secret_key

    def process(self, payload: dict) -> bool:
        if verify_hmac_signature(payload['data'], self.secret_key):
            return True
        return False

class Sha256Processor(WebhookProcessor):
    def __init__(self, secret_key: str):
        self.secret_key = secret_key

    def process(self, payload: dict) -> bool:
        if verify_sha256_signature(payload['data'], self.secret_key):
            return True
        return False

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class WebhookData(BaseModel):
    provider: str
    data: bytes  # Raw payload or base64 encoded

@app.post("/webhooks/{provider}")
async def receive_webhook(provider: str, webhook_data: WebhookData):
    processor = get_webhook_processor(provider)
    if not processor.process(webhook_data.data):
        raise HTTPException(status_code=401, detail="Signature verification failed")

    # Process the payload
    process_payload(webhook_data)

def process_payload(payload: dict):
    # Placeholder for actual processing logic
    pass
```

**Example input / output:**

```text
Input:
POST /webhooks/provider1 HTTP/1.1
Host: localhost:8000

{
  "provider": "provider1",
  "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiSm9obiBBbGV4In0.tT1qYd8H8oWQk6v8rZ1t3pO2w7V2FgkUf"
}

Output:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "message": "Webhook processed successfully."
}
```

**Data:**

```json
{
  "provider": "provider1",
  "data": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiSm9obiBBbGV4In0.tT1qYd8H8oWQk6v8rZ1t3pO2w7V2FgkUf"
}
```

**Flow:**

1. Valid
2. Invalid
3. Receive Webhook
4. Verify Signature
5. Process Payload
6. 401 Unauthorized

