---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between HTTP status codes 401 and 403?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ HTTP Status Codes](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Status_Codes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with authentication and authorization in web applications, understanding the differences between HTTP status codes 401 (Unauthorized) and 403 (Forbidden) is crucial. These two codes help you communicate to the client what went wrong when they tried to access a resource.

#### Triaging the Question

The question asks for the difference between 401 Unauthorized and 403 Forbidden HTTP status codes. The interviewer is probing your understanding of REST API security, HTTP responses, and server-client communication patterns.

#### Assumptions
We assume these responses are part of an HTTP interaction.
The client has attempted to access a protected resource but encountered different response behaviors due to authentication or authorization issues.

#### First-Principles Solution
401 Unauthorized:

This status code indicates that the request requires user authentication. It is typically sent in response to a request that contains bad credentials (such as an invalid username and password) or no credentials at all. Upon receiving a 401, the client should send back its credentials using proper methods like HTTP Basic Authentication, Digest Authentication, or OAuth.

403 Forbidden:

This status code means the server understood the request but refuses to authorize it. In contrast to 401, this indicates that even if the user was authenticated correctly (i.e., sent valid credentials), they do not have access rights to perform the requested action on a resource. This could be due to roles-based access control policies or insufficient permissions.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, 401 Unauthorized is about authentication issues where the client needs to provide proper credentials, while 403 Forbidden addresses authorization issues when authenticated users lack permission for a particular resource. Understanding these differences helps in handling different types of HTTP errors effectively.

#### Common mistakes
include confusing both statuses or not properly using them based on the specific situation. For example, sending a 401 instead of 403 when the user is already authenticated and simply does not have access rights can lead to usability issues for the client.

If asked further, you might discuss how these status codes affect user experience, API design best practices, and security considerations.

**Code:**

```python
# Example of handling HTTP 401 Unauthorized in a Flask application
@app.route('/protected')
def protected():
    auth = request.authorization
    if not auth or not check_credentials(auth.username, auth.password):
        return jsonify({"error": "Unauthorized"}), 401

    # User is authenticated, proceed with access control checks
    user = get_user_by_username(auth.username)
    if not has_permission(user.id, 'view_protected_data'):
        return jsonify({"error": "Forbidden"}), 403

    return jsonify({"message": "Access granted!"})

# Example of handling HTTP 403 Forbidden in a Django application
def protected_view(request):
    user = authenticate(request)
    if not user or not check_credentials(user.username, request.password):
        return HttpResponse(status=401)

    # User is authenticated, proceed with access control checks
    if not has_permission(user.id, 'view_protected_data'):
        return HttpResponse("Forbidden", status=403)

    return JsonResponse({"message": "Access granted!"})
```

**Example input / output:**

```text
Example for Unauthorized Response:
Request Headers: Authorization: Basic Zm9vOmJhcg==
HTTP Response:
Status Code: 401
Content-Type: application/json
Response Body: {"error": "Unauthorized"}

Example for Forbidden Response:
Request Headers: Authorization: Basic a2V5OmRhc2s=
HTTP Response:
Status Code: 403
Content-Type: text/plain
Response Body: Forbidden
```

**Data:**

```json
{
  "example_request_unauthorized": {
    "method": "GET",
    "url": "/protected",
    "headers": {
      "Authorization": "Basic Zm9vOmJhcg=="
    },
    "auth_status": 401,
    "message": "Unauthorized"
  },
  "example_request_forbidden": {
    "method": "GET",
    "url": "/protected",
    "headers": {
      "Authorization": "Basic a2V5OmRhc2s="
    },
    "auth_status": 200, // Assume user is authenticated
    "access_status": 403,
    "message": "Forbidden"
  }
}
```

