---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks how you would prevent a single leaked API token from exfiltrating your entire customer database. What scoping, egress, and detection layers would you put in place?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Secrets and Key Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To prevent a single leaked API token from compromising an entire customer database, we need to implement a layered security approach that covers both prevention, detection, and response. At a high level, this involves:

Scope and Limit Access: Define the scope of access for each API token.
Egress Control: Monitor and control data egress (data leaving your systems).
Detection and Response: Implement mechanisms to detect and respond to suspicious activities.

This layered approach ensures that even if a single token is leaked, the damage can be contained and mitigated effectively.

**Comparison:**

```
Layer	Description
Scope	Define and enforce the scope of each API token to ensure it only accesses allowed resources.
Egress Control	Monitor data egress by tracking where and how customer data is leaving your system, with logging and alerts.
Detection & Response	Implement real-time detection mechanisms for suspicious activities, along with automated and manual response procedures.
```

**Code:**

```python
# Example of code to set up logging for API calls with rate limiting
from flask import Flask, request
import requests

app = Flask(__name__)

@app.route('/api/data', methods=['GET'])
def get_data():
    token = request.headers.get('X-Api-Token')

    # Check if the token is valid and within scope
    if not validate_token(token):
        return "Invalid or unauthorized API token", 401

    try:
        # Simulate data retrieval from a database
        customer_data = get_customer_data()

        # Log details of the request
        log_request_details(request, customer_data)

        return customer_data
    except Exception as e:
        app.logger.error(f"Error retrieving data: {e}")
        return "An error occurred while fetching data", 500

def validate_token(token):
    # Validate and scope the token here
    pass

def get_customer_data():
    # Simulate database query or API call
    pass

def log_request_details(request, response_data):
    # Log relevant details for auditing purposes
    app.logger.info(f"API Token: {token[:4]}... | Requested by IP: {request.remote_addr} | Response Size: {len(response_data)} bytes")
```

**Example input / output:**

```text
Input:
- API request with a valid token but an unauthorized scope, e.g., /customer/1234 with the '67890' token

Output:
- Log entry indicating invalid or unauthorized token access attempt
- Response with 401 Unauthorized status code
```

**Data:**

```json
{
  "api_tokens": [
    {
      "id": "12345",
      "scope": "/customer/*",
      "description": "Read-only access to customer data"
    },
    {
      "id": "67890",
      "scope": "/internal/",
      "description": "Access for internal operations"
    }
  ]
}
```

**Flow:**

1. Valid
2. Invalid
3. API Token Request
4. Validate Token
5. Process Request
6. Log & Respond
7. Retrieve Customer Data
8. Log Request Details
9. Return Response
10. Alert Security Team
11. Manual Intervention
12. Contain and Remediate
13. Egress Monitoring

