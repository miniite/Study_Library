---
type: "practice"
difficulty: "Expert"
---

# How would you implement refresh-token rotation with HTTP-only cookies in a FastAPI authentication system?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Authentication](<../../../../TOOLS/Python/FastAPI/API_Authentication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure secure and efficient user sessions using FastAPI by implementing refresh-token rotation with HTTP-only cookies. This involves understanding how session management works in web applications, ensuring that tokens are securely handled, and leveraging FastAPI's capabilities.

Firstly, let's briefly outline the key components involved:

User Authentication: The initial login request generates a pair of access and refresh tokens.
Access Token Usage: Each API call made by the user includes the access token in the Authorization header.
Refresh Token Handling: When an access token expires, it is replaced with a new one using the refresh token stored in an HTTP-only cookie.

Now, let's dive into the implementation:

#### Triage
Understanding the Problem and Probing

The question probes your understanding of:

Secure handling of tokens (access and refresh)

#### Implementation
of HTTP-only cookies
Token rotation mechanisms
Best practices for session management

We will assume that we are working with a FastAPI application, and we need to ensure that user sessions are managed securely while providing efficient token handling.

#### First-Principles Solution
Step-by-Step Breakdown
Token Generation: Generate both access and refresh tokens upon login.
Access Token Usage: Use the access token for API requests.
Refresh Token Storage: Store the refresh token in an HTTP-only cookie to prevent client-side tampering.
Token Rotation Mechanism: Implement a mechanism to rotate tokens when needed, ensuring that the user's session remains uninterrupted.
Step 1: Token Generation

Upon successful login, we generate both access and refresh tokens:

PYTHON
from fastapi import Depends, HTTPException
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "supersecretkey"
ALGORITHM = "HS256"

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=10)  # Example expiration time
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def create_refresh_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(days=30)  # Example expiration time
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
Step 2: Access Token Usage

For API requests, the access token is included in the Authorization header:

PYTHON
from fastapi import FastAPI, Depends, HTTPException
import jwt

app = FastAPI()

async def get_current_user(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token has expired")

@app.get("/api/v1/protected")
async def read_protected_content(current_user: dict = Depends(get_current_user)):
    return {"message": "Welcome to the protected content!"}
Step 3: Refresh Token Storage

Store the refresh token in an HTTP-only cookie:

PYTHON
from fastapi import Cookie, Response

@app.post("/login")
async def login(response: Response):
    # Simulate successful login and generate tokens
    access_token = create_access_token({"sub": "user_id"})
    refresh_token = create_refresh_token({"sub": "user_id"})

    response.set_cookie(key="refresh_token", value=refresh_token, httponly=True)
    return {"access_token": access_token}
Step 4: Token Rotation Mechanism

When an access token expires, it should be replaced using the refresh token. We can implement a middleware to handle this:

PYTHON
from fastapi import FastAPI, HTTPException
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "supersecretkey"
ALGORITHM = "HS256"

def check_and_rotate_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        if "exp" in payload and datetime.utcnow() > payload["exp"]:
            # Token has expired, need to rotate
            refresh_token = request.cookies.get("refresh_token")
            if not refresh_token:
                raise HTTPException(status_code=401, detail="Refresh token missing")

            try:
                new_payload = jwt.decode(refresh_token, SECRET_KEY, algorithms=[ALGORITHM])
                access_token = create_access_token(new_payload)
                return {"access_token": access_token}
            except jwt.ExpiredSignatureError:
                raise HTTPException(status_code=401, detail="Refresh token has expired")
        else:
            # Token is still valid
            return payload
    except jwt.DecodeError:
        raise HTTPException(status_code=401, detail="Invalid token")

@app.middleware("http")
async def check_token(request: Request, call_next):
    access_token = request.headers.get("Authorization", "").split(" ")[1]
    response = await check_and_rotate_token(access_token)
    if isinstance(response, dict):  # Token rotation successful
        return Response(status_code=200, content={"access_token": response["access_token"]})
    else:
        return await call_next(request)

#### Closing Summary
and Common Mistakes

In summary, we have implemented a secure authentication system using FastAPI that includes token generation, usage, storage in HTTP-only cookies, and efficient rotation mechanisms. Key points include:

Secure Token Handling: Using JWT for both access and refresh tokens.
HTTP-Only Cookies: Ensuring client-side tampering is minimized by storing sensitive information in cookies with the httponly flag.
Token Rotation: Implementing a middleware to handle token expiration seamlessly.

#### Common mistakes
include:

Not handling token expiration properly, leading to session issues.
Failing to use HTTP-only cookies for sensitive tokens, exposing them to client-side attacks.
Lack of proper validation and error handling in the token rotation mechanism.

**Code:**

```python
from fastapi import Depends, HTTPException
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "supersecretkey"
ALGORITHM = "HS256"

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=10)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def create_refresh_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(days=30)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def check_and_rotate_token(token: str):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        if "exp" in payload and datetime.utcnow() > payload["exp"]:
            refresh_token = request.cookies.get("refresh_token")
            if not refresh_token:
                raise HTTPException(status_code=401, detail="Refresh token missing")

            try:
                new_payload = jwt.decode(refresh_token, SECRET_KEY, algorithms=[ALGORITHM])
                access_token = create_access_token(new_payload)
                return {"access_token": access_token}
            except jwt.ExpiredSignatureError:
                raise HTTPException(status_code=401, detail="Refresh token has expired")
        else:
            return payload
    except jwt.DecodeError:
        raise HTTPException(status_code=401, detail="Invalid token")

@app.middleware("http")
async def check_token(request: Request, call_next):
    access_token = request.headers.get("Authorization", "").split(" ")[1]
    response = await check_and_rotate_token(access_token)
    if isinstance(response, dict):  # Token rotation successful
        return Response(status_code=200, content={"access_token": response["access_token"]})
    else:
        return await call_next(request)

@app.post("/login")
async def login(response: Response):
    access_token = create_access_token({"sub": "user_id"})
    refresh_token = create_refresh_token({"sub": "user_id"})
    response.set_cookie(key="refresh_token", value=refresh_token, httponly=True)
    return {"access_token": access_token}

@app.get("/api/v1/protected")
async def read_protected_content(current_user: dict = Depends(get_current_user)):
    return {"message": "Welcome to the protected content!"}
```

**Flow:**

1. Yes
2. No
3. User logs in
4. Generate access and refreshtokens
5. Set cookies with refresh token(HTTP-only)
6. Access token is used for APIrequests
7. Check if access token expired
8. Access Token Expiration
9. Use refresh token to get newaccess token
10. Return new access token andrefresh token
11. Continue with request usingcurrent access token
12. Handle requests securely

