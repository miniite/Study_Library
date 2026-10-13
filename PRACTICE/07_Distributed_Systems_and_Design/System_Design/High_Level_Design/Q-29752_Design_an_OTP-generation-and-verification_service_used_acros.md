---
type: "practice"
difficulty: "Intermediate"
---

# Design an OTP-generation-and-verification service used across products — how do you rate-limit per number resend prevent enumeration and store OTPs securely with short TTL at scale?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiter Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust OTP (One-Time Password) generation, verification, and storage system, we need to address several key aspects: rate limiting per phone number, preventing enumeration attacks, secure storage with short Time-To-Live (TTL), and scaling the service efficiently. The goal is to ensure that the system is both secure and performant.

First, let's break down what the interviewer might be probing:

Rate Limiting: Ensuring users can't spam requests for OTPs.
Preventing Enumeration Attacks: Protecting against attackers trying to guess phone numbers by sending OTPs and verifying them.
Secure Storage: Storing OTPs securely with a short TTL to minimize exposure.
Scalability: Designing the system to handle high traffic without compromising performance or security.

#### Assumptions
The system will be used across multiple products, so it should be modular and reusable.
OTPs are 6-digit numeric values.
The system needs to support different time zones.
The service should have a mechanism for logging and monitoring for auditing purposes.
Step-by-Step Solution
1. Rate Limiting

To rate-limit OTP requests, we can implement a sliding window approach:

Rate-Limiting Strategy: Each phone number will be allowed to receive a maximum of 5 OTPs in a 30-minute period.

#### Implementation
Maintain an in-memory cache (e.g., Redis) to track the number of requests per phone number and time.
2. Preventing Enumeration

To prevent enumeration, we can:

Masked Phone Numbers: Use masked phone numbers in logs to avoid exposing actual phone numbers.
Limiter Implementation:
When a request is made for an OTP, check if the phone number has already been used recently.
If not, generate and send the OTP; otherwise, reject the request.
3. Secure Storage

To securely store OTPs with a short TTL:

OTP Generation: Generate OTPs using a cryptographically secure random number generator.
TTL Implementation:
Store each OTP in an in-memory cache (e.g., Redis) with a TTL of, say, 5 minutes.
Use a database like PostgreSQL for storing historical OTP attempts and their results.
4. Scalability

To ensure the system scales:

Microservices Architecture: Break down the system into microservices to handle different parts of the process (e.g., OTP generation, storage, verification).
Load Balancing: Use load balancers to distribute traffic across multiple instances.
Caching: Utilize caching mechanisms like Redis for storing and validating OTPs.

#### Closing Summary

By implementing a rate-limiting strategy, preventing enumeration attacks, securely storing OTPs with short TTLs, and ensuring the system is scalable, we can design a robust OTP service that meets the required security and performance standards. The key components include:

Rate-Limiting: Using in-memory caching.
Preventing Enumeration: Masked phone numbers and limited request frequency.
Secure Storage: Caching with TTL and database storage.

#### Common Mistakes
Not Implementing Rate Limiting: This can lead to abuse of the service.
Failing to Mask Phone Numbers in Logs: Exposing sensitive information.
Using a Single Long-Term Cache for OTPs: Increasing risk of exposure if the cache is compromised.

**Comparison:**

```
Component	Description
Rate Limiting	Use a Redis cache to track and limit the number of OTP requests per phone number.
Prevent Enumeration	Mask phone numbers in logs and enforce rate limits.
Secure Storage	Store OTPs in Redis with TTL and use PostgreSQL for historical records.
Scalability	Design as microservices, use load balancers, and caching mechanisms.
```

**Code:**

```python
import redis
from datetime import timedelta

# Initialize Redis client
redis_client = redis.StrictRedis(host='localhost', port=6379, db=0)

def send_otp(phone_number):
    # Check rate limit
    key = f"rate_limit:{phone_number}"
    if not redis_client.exists(key) or int(redis_client.get(key)) < 5:
        # Increment the counter for the phone number
        redis_client.incr(key)
        redis_client.expire(key, timedelta(minutes=30))

        # Generate and send OTP (mock function)
        otp = generate_otp()
        send_sms(phone_number, otp)

        return {"status": "OTP sent"}
    else:
        return {"status": "Rate limit exceeded"}

def validate_otp(phone_number, otp):
    key = f"validate:{phone_number}:{otp}"
    if redis_client.exists(key):
        # OTP is already validated
        return {"status": "OTP already used"}

    # Check and validate OTP (mock function)
    if verify_otp(phone_number, otp):
        # Invalidate the OTP to prevent reuse
        redis_client.delete(key)
        return {"status": "OTP verified successfully"}
    else:
        return {"status": "Invalid OTP"}
```

**Example input / output:**

```text
# Example input and output for send_otp
input = {"phone_number": "+1234567890"}
output = send_otp(input["phone_number"])
print(output)  # {"status": "OTP sent"}

# Example input and output for validate_otp
input = {"phone_number": "+1234567890", "otp": "123456"}
output = validate_otp(input["phone_number"], input["otp"])
print(output)  # {"status": "OTP verified successfully"}
```

**Data:**

```json
{
  "phone_number": "+1234567890",
  "otp": "123456"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. User Request
8. Rate Limit Check
9. Is Rate Limit Exceeded?
10. Reject Request
11. Generate OTP
12. Send SMS with OTP
13. OTP Validation
14. Is OTP Valid?
15. Mark OTP as Used
16. Reject Request
17. Validate OTP Request
18. Is OTP Already Used?
19. Reject Request
20. Verify OTP
21. Mark OTP as Validated &Invalidate

