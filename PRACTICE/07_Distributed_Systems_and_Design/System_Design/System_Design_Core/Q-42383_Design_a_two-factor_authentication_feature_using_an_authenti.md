---
type: "practice"
difficulty: "Intermediate"
---

# Design a two-factor authentication feature using an authenticator app.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Notification System](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Notification_System.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
To design a two-factor authentication (2FA) feature using an authenticator app, we need to ensure that our system not only provides strong security but also offers a seamless user experience. The core idea is to generate time-based one-time passwords (TOTPs) on the client side and validate them against a server-side secret key.

#### Triage
This question aims to evaluate your understanding of authentication mechanisms, the trade-offs between security and usability, and your ability to design a scalable system. Key assumptions include that we are using an industry-standard algorithm like TOTP and that the user's device has access to a trusted authenticator app. The main steps involve generating and validating TOTP tokens.

#### Solution

User Registration:

When a new user registers, their account is created on the server with a unique identifier (UUID) and a randomly generated secret key.
This secret key is then sent to the user's authenticator app via QR code or a URL in plaintext for simplicity.

Token Generation:

The authenticator app generates TOTP tokens based on the shared secret key and a time-based function (e.g., using the HOTP algorithm).
These tokens are valid for a short duration, typically 30 seconds, to ensure they can only be used in real-time.

Token Validation:

When a user attempts to log in, they enter their username or email and the current TOTP token from their authenticator app.
The server generates the expected TOTP for that time period using the stored secret key.
If the entered token matches the expected one, the authentication is successful; otherwise, it fails.

Security Considerations:

Use HTTPS to encrypt data in transit between the client and server.
Implement rate limiting on login attempts to prevent brute-force attacks.
Store only minimal metadata about TOTP tokens for logging purposes, if needed.

Usability Enhancements:

Provide a user-friendly interface for managing authenticator apps (e.g., generating QR codes).
Allow users to generate backup tokens in case their primary device fails.

Scalability and Performance:

Optimize token generation on the client side to minimize server load.
Implement caching mechanisms to reduce database access frequency.

#### Closing Summary
This 2FA design ensures a balance between security and usability by leveraging TOTP-based authentication. By following best practices in both implementation and user experience, we can create a robust system that protects against common attacks while providing a seamless login process for users.

#### Common Mistakes/Follow-Ups

Forgetting to use encryption when storing or transmitting the secret key.
Not implementing rate limiting, leading to potential brute-force attacks.
Ignoring user experience by making the setup and management of 2FA overly complicated.
Failing to account for edge cases like device loss or password resets.

**Comparison:**

```
Authentication Method	Security Level	Usability
2FA (TOTP)	High	Moderate
```

**Code:**

```python
from datetime import datetime
import pyotp

# User registration (simplified)
def register_user(user_id):
    secret_key = pyotp.random_base32()
    return {"user_id": user_id, "secret_key": secret_key}

# Token generation in the authenticator app
def generate_totp(secret_key: str) -> str:
    totp = pyotp.TOTP(secret_key)
    return totp.now()

# Token validation on the server
def validate_totp(user_id, input_token):
    user_info = get_user_by_id(user_id)
    if not user_info:
        return False
    secret_key = user_info["secret_key"]
    totp = pyotp.TOTP(secret_key)
    return totp.verify(input_token)
```

**Example input / output:**

```text
Input:
User attempts to log in with email: test@example.com and TOTP token: JBSWY3DPEHPK3PXP

Expected Output:
Authentication successful.
```

**Data:**

```json
{
  "user": {
    "user_id": "1234567890abcdef",
    "email": "test@example.com",
    "secret_key": "JBSWY3DPEHPK3PXP"
  }
}
```

**Flow:**

1. QR Code or URL
2. Match
3. Mismatch
4. User Registration
5. Generate TOTP Secret
6. Client
7. Token Generation
8. Server Login Request
9. Validate TOTP
10. Authentication Success
11. Authentication Failure

