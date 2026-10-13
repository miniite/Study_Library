---
type: "practice"
difficulty: "Intermediate"
---

# How would you secure your own AI chatbot REST API so only authenticated users can call the expensive LLM endpoints?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling 429 Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Handling_429_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that only authenticated users can access the expensive Large Language Model (LLM) endpoints in an AI chatbot's REST API, we need to implement robust authentication and authorization mechanisms. This involves understanding the different types of authentication methods available and their pros and cons. Let’s break down the solution step by step.

First, it's important to clarify what we are trying to achieve: secure our LLM endpoints so that only authenticated users can access them, thus preventing unauthorized calls and managing costs effectively. The key is to implement a token-based system where each user must have an active token to make valid requests.

#### Triage

The question essentially asks how to restrict API access based on authentication status. We need to consider the following:

Authentication Mechanism: How will we identify users?
Token Management: What kind of tokens should we use and how do they work?
Authorization: Once authenticated, what permissions does a user have?

#### Assumptions
Users are identified by unique credentials (e.g., email, username).
We have an existing system to manage these credentials.
We can implement custom middleware in our API server.

#### First-Principles Solution
1. Implement Token-Based Authentication

We will use JWT (JSON Web Tokens) for authentication because they are lightweight and easy to transmit over the wire. Here's a high-level flow:

PLAINTEXT
User -> [Login] -> Server: Verify credentials -> Generate JWT -> Return JWT to User
2. Protect LLM Endpoints with Middleware

We will implement middleware in our API server that checks for valid tokens before allowing access to LLM endpoints.

3. Token Validation and Expiration
Validation: The middleware should validate the token by checking its signature and claims.
Expiration: Tokens should expire after a certain period, requiring users to log in again.
4. Custom Middleware Implementation (Example)

Here is an example of how we can implement this using Python with Flask:

PYTHON
from flask import request, jsonify
from functools import wraps
import jwt
import time

# Example secret key for JWT generation and verification
SECRET_KEY = 'your_secret_key'

def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = None
        if 'Authorization' in request.headers:
            token = request.headers['Authorization'].split()[1]

        if not token:
            return jsonify({'message': 'Token is missing!'}), 403

        try:
            data = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
            current_user = get_user_from_db(data['user_id'])  # Custom function to fetch user from DB
        except Exception as e:
            print(e)
            return jsonify({'message': 'Token is invalid!'}), 403

        return f(current_user, *args, **kwargs)

    return decorated

@token_required
def llm_endpoint(current_user):
    # LLM API call logic here
    pass
5. Additional Measures for Security
Rate Limiting: To prevent abuse, we can implement rate limiting on the number of requests per user.
HTTPS: Ensure all communication is over HTTPS to protect tokens in transit.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, securing our AI chatbot's LLM endpoints involves implementing JWT-based token authentication and protecting these endpoints with middleware. We must ensure that tokens are valid, secure, and have proper expiration policies. Common mistakes include not validating tokens properly or not handling expired tokens gracefully.

For follow-ups, the interviewer might ask about:

How to handle cases where a user's session is compromised.
The impact of token expiration on user experience.
Alternative authentication methods if JWT isn't suitable for your use case.

