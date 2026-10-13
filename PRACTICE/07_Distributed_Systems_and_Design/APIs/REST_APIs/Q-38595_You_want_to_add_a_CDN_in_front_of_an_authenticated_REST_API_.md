---
type: "practice"
difficulty: "Deep"
---

# You want to add a CDN in front of an authenticated REST API. What must be true about cache keys, Vary headers, and cookie handling to make it safe, and which endpoints do you deliberately never cache?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ HTTP Caching and ETags](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Caching_and_ETags.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To safely integrate a Content Delivery Network (CDN) in front of an authenticated REST API, we must carefully consider how cache keys are constructed, the use of Vary headers, and cookie handling. The goal is to ensure that cached content cannot be shared between different users or sessions while maintaining performance benefits from CDN caching. Specifically, certain endpoints should never be cached at all.

Summary: To make an authenticated REST API safe with a CDN, we need to ensure cache keys are unique per user session, Vary headers appropriately control caching based on cookies and request parameters, and critical authentication-related endpoints are excluded from caching entirely.

#### Triage
The question is probing the fundamental concepts of HTTP caching and how they interact with authentication. We must assume that the backend API uses session-based authentication (e.g., JWT) and employs standard HTTP methods like GET, POST, PUT, DELETE. Our solution will focus on key headers and cookies that need to be considered.

#### First Principles Solution

Cache Keys: Unique cache keys for each user or session are essential. This can be achieved by incorporating a unique identifier (e.g., JWT token) into the cache key.
Vary Headers: Using Vary headers correctly is critical. For instance, if a GET request relies on cookies to authenticate, we need to set Vary: Cookie. If there are other parameters that change the response, like pagination or filtering queries, include those as well in the Vary header.
Cookie Handling: Authentication cookies should not be included in cached responses. This ensures that each user's session is isolated.

Endpoints to Never Cache:

All endpoints that involve user authentication (e.g., login, logout, token refresh).
Endpoints that return sensitive or personalized data based on the user’s session.
Any endpoint that could lead to security vulnerabilities if cached improperly.

Let's break this down step-by-step:

Cache Keys:

#### Cons
truct cache keys using a combination of the request path and relevant non-cookie headers, but also include unique identifiers from cookies (e.g., JWT).
PYTHON
cache_key = f"{request.path}{request.headers['Authorization']}"

Vary Headers:

Include Cookie in Vary headers to ensure that caching respects the authentication state.
PYTHON
response.headers["Vary"] = "Cookie"

Cookie Handling:

Set appropriate HTTP-only and secure flags on authentication cookies.
Ensure no sensitive data is sent in cached responses by checking if the response contains any Set-Cookie headers that should be excluded from caching.

Endpoints to Never Cache:

For example, endpoints like /login, /logout, /refresh_token, and any endpoint returning user-specific or session-bound information (e.g., /user_profile/{userId}).
PYTHON
if request.path in ["/login", "/logout", "/refresh_token"]:
    response.headers["Cache-Control"] = "no-cache, no-store, must-revalidate"
    response.headers["Pragma"] = "no-cache"
    response.headers["Expires"] = "0"

#### Closing Summary
By carefully managing cache keys, Vary headers, and cookie handling, we can safely integrate a CDN with an authenticated REST API. Critical endpoints are excluded from caching to prevent security risks while still benefiting from performance improvements.

#### Common Mistakes/Follow-Ups

Failing to include the Cookie in the Vary header.
Not setting appropriate cache control headers for critical endpoints.
Ignoring session-based authentication when constructing cache keys and varying headers.
Misconfiguring cookie flags (e.g., not marking cookies as HTTP-only or secure).

**Comparison:**

| Endpoint | Cache Control Headers |
|----------|-----------------------|
| /login   | no-cache, no-store, must-revalidate |
| /logout  | no-cache, no-store, must-revalidate |
| /refresh_token | no-cache, no-store, must-revalidate |

**Code:**

```python
# Example of setting appropriate cache control headers in a Flask app
@app.route('/login', methods=['POST'])
def login():
    # ... authentication logic ...
    response = make_response(jsonify({'message': 'Login successful'}), 200)
    response.headers["Cache-Control"] = "no-cache, no-store, must-revalidate"
    response.headers["Pragma"] = "no-cache"
    response.headers["Expires"] = "0"
    return response
```

**Example input / output:**

```text
# Example of a dry-run for setting cache control headers
def handle_request(request):
    if request.path in ["/login", "/logout"]:
        response = make_response(jsonify({'message': 'Endpoint should never be cached'}), 200)
        response.headers["Cache-Control"] = "no-cache, no-store, must-revalidate"
        response.headers["Pragma"] = "no-cache"
        response.headers["Expires"] = "0"
    else:
        # Construct cache key
        cache_key = f"{request.path}{request.headers['Authorization']}"

        # Set Vary headers if needed
        response.headers["Vary"] = "Cookie"

        # Return the constructed response
        return response

# Input: Request for /login
input_request = {
    "path": "/login",
    "headers": {"Authorization": "Bearer JWT_TOKEN"}
}

# Expected Output:
expected_response = make_response(jsonify({'message': 'Endpoint should never be cached'}), 200)
expected_response.headers["Cache-Control"] = "no-cache, no-store, must-revalidate"
expected_response.headers["Pragma"] = "no-cache"
expected_response.headers["Expires"] = "0"
```

**Data:**

```json
{
  "cache_key": "/user_profile/12345?sort=desc&filter=newest",
  "request_headers": {
    "Authorization": "Bearer JWT_TOKEN",
    "Cookie": "auth=secure_token; other_cookie=value"
  },
  "vary_header": ["Cookie"]
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[User requests API] --> B[API checks request path for special endpoints];
4. B --> C[Returns 403 or 401 and sets cache control headers to "no-cache"];
5. B --> D[API continues processing, constructs unique cache key using JWT token];
6. D --> E[Sets Vary header with "Cookie" if necessary];
7. D --> F[Returns response from API];

