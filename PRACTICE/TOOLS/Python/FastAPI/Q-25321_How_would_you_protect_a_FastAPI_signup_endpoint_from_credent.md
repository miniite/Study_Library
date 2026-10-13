---
type: "practice"
difficulty: "Deep"
---

# How would you protect a FastAPI signup endpoint from credential-stuffing and enumeration attacks while keeping legitimate latency low, and what signals feed your defense?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Authentication](<../../../../TOOLS/Python/FastAPI/API_Authentication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To protect a FastAPI signup endpoint from credential-stuffing and enumeration attacks while maintaining low legitimate latency, we need to implement robust defensive strategies that can distinguish between automated brute-force attempts and genuine user signups. This involves leveraging rate limiting, CAPTCHAs, and behavioral analysis to dynamically adjust our defenses based on the incoming traffic's characteristics.

The key idea is to balance security with performance by implementing a layered defense strategy that starts from simple rate limiting mechanisms and escalates as suspicious behavior is detected. We'll start by understanding the core problem: credential stuffing involves automated tools trying multiple usernames/passwords combinations, while enumeration attacks aim to guess valid email addresses or usernames using a combination of social engineering and brute-force methods.

**Comparison:**

```
Rate Limiting	Behavior Analysis
Prevents rapid attempts from single IPs or user agents.	Analyzes the time between signups, keyboard typing patterns, etc., to flag suspicious behavior.
```

**Code:**

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
import models, schemas, utils

router = APIRouter()

# Rate Limiting Middleware
def rate_limit_middleware(get_response):
    # Implementation of ratelimit logic using Redis or another backend
    pass

@router.post("/signup", dependencies=[Depends(rate_limit_middleware)])
async def signup(user: schemas.UserCreate, db: Session = Depends(models.get_db)):
    # Check if email already exists in the database
    user_in_db = db.query(models.User).filter(models.User.email == user.email).first()
    if user_in_db:
        raise HTTPException(status_code=409, detail="Email already registered")

    # Verify CAPTCHA response (if needed)
    await verify_captcha(user.captcha_response)

    # Create a new user
    new_user = models.User(email=user.email, password=utils.hash_password(user.password))
    db.add(new_user)
    db.commit()
    db.refresh(new_user)

    return {"message": "User created successfully"}
```

**Example input / output:**

```text
Input:
POST /signup
{
  "email": "user@example.com",
  "password": "securepassword123!"
}

Output:
{
  "message": "User created successfully"
}
```

**Data:**

```json
{
  "UserCreate": {
    "email": "user@example.com",
    "password": "securepassword123!",
    "captcha_response": "true"
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[User Signs Up] --> B[Ratelimit Check];
4. B --> C[Captcha Verification];
5. C --> D[Behavior Analysis];
6. D --> E[Create User];
7. E --> F[Success Response];
8. B fail --> G[Block Request];
9. D fail --> H[Rate Limit Exceeded];

