---
type: "practice"
difficulty: "Intermediate"
---

# What is CORS, and why do browsers enforce it when calling REST APIs?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CORS](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

CORS (Cross-Origin Resource Sharing) is a security mechanism that restricts how a web page served from one origin can interact with resources from another origin. Browsers enforce CORS to prevent malicious websites from making unauthorized requests on behalf of the user to a server they are not supposed to access.

To understand why browsers enforce CORS, we need to delve into the basics of how HTTP and web pages work. When a web page is loaded in a browser, it is considered to be served from a specific origin (a combination of protocol, domain name, and port). For example, if a webpage is accessed at https://example.com, that's its origin.

Browsers enforce CORS because without such restrictions, a malicious website could inject JavaScript that attempts to make HTTP requests to other origins. This could potentially lead to security vulnerabilities like session hijacking or accessing sensitive data from another site. For instance, if attacker.com tries to steal data from target.com, it can only do so by injecting JavaScript on attacker.com and then sending HTTP requests to target.com. With CORS enabled, this behavior is blocked unless the server explicitly allows such cross-origin requests.

Now let's break down the mechanics of how browsers enforce CORS:

Origin Headers: When a client (web browser) makes an HTTP request, it includes several headers in its request. One crucial header that plays a role here is the Origin header, which specifies the origin from where the request was made.

CORS Policy Enforcement: The server must respond with appropriate CORS headers to allow cross-origin requests. If these headers are missing or incorrectly configured, the browser will block the request due to security concerns.

Access-Control-Allow-Origin Header: This header is a key part of CORS policy enforcement. It specifies which origins have permission to access the resource. For example:

HTTP
Access-Control-Allow-Origin: https://example.com

CORS Preflight Requests (OPTIONs Method): In certain cases, when the request involves non-simple methods (e.g., PUT, DELETE) or specific headers that aren't allowed by default (like Content-Type), browsers will perform an additional OPTIONs request before making the actual request. This preflight request asks for permission to make the real request.

Now let's illustrate this with a simple example:

#### Example
A Request from attacker.com

Client Request: A script on attacker.com tries to send an HTTP GET request to target.com/api/data.

Request Headers:

HTTP
Origin: https://attacker.com

Response from target.com: If the server does not include proper CORS headers, it will respond with a status code indicating failure (e.g., 403 Forbidden).

#### Example
of Proper CORS Response
Client Request: Same as above.
Server Response:
HTTP
Access-Control-Allow-Origin: https://attacker.com

If this header is present and matches the Origin header, the request will proceed.

#### Common mistakes
in implementing CORS include not including the Access-Control-Allow-Origin header at all or having it set incorrectly. Additionally, developers often overlook preflight OPTIONs requests for more complex scenarios.

In conclusion, CORS is essential for maintaining security and preventing malicious websites from making unauthorized HTTP requests to servers they shouldn't access. Properly configuring CORS policies in your server-side code ensures that only trusted origins can interact with your API endpoints.

**Code:**

```
HTTP
Copy
# Example of a proper CORS response header
Access-Control-Allow-Origin: https://example.com
```

**Flow:**

1. Origin Request
2. Browser Adds Origin Header
3. Server Receives Request withOrigin Header
4. Server Checks CORS Policy
5. Server Responds with CORSHeaders, if any
6. Browser Validates CORSHeaders and Proceeds orBlocks Request

