---
type: "practice"
difficulty: "Deep"
---

# What are the trade-offs of storing a JWT in a cookie versus localStorage from both a security and usability standpoint?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Keys vs Tokens](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Keys_vs_Tokens.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Storing a JSON Web Token (JWT) in either a cookie or localStorage involves different trade-offs, particularly in terms of security and usability. Let's break down these considerations step by step.

#### Triaging the Question

The core of this question revolves around understanding how JWTs are stored and their implications on both user experience and security. Specifically, we need to compare cookies with localStorage.

Security: How does each storage mechanism handle token protection?
Usability: How do these methods affect the overall functionality and performance of the application?

#### Assumptions
We assume JWTs are used for stateless authentication.
Both cookies and localStorage can store JSON data.
The application is a web app that requires user sessions to be maintained.
Step-by-Step Solution
1. Security Considerations

Cookies:

HTTP Only Flag: When using cookies, you can set the HttpOnly flag to prevent client-side scripts from accessing them directly.
Secure Flag: Setting the Secure flag ensures that the cookie is only transmitted over HTTPS connections, enhancing security.
SameSite Attribute: The SameSite attribute can be set to Strict or Lax, which prevents cookies from being sent in cross-site request forgery (CSRF) attacks.

LocalStorage:

Client-Side Accessible: Since the JWT is stored in localStorage, client-side JavaScript can directly access and manipulate it.
No Direct Protection Mechanisms: Unlike cookies, there are no built-in mechanisms to prevent direct access by client-side code.
2. Usability Considerations

Cookies:

Cross-Domain Issues: Cookies associated with a domain cannot be read or manipulated on another domain without proper configuration (using Access-Control-Allow-Credentials).
Performance Overhead: Sending cookies over HTTP/HTTPS adds additional overhead, but this is usually negligible for most applications.

LocalStorage:

Simpler Implementation: Storing JWTs in localStorage is straightforward and doesn't affect the server's response headers.
Client-Side Flexibility: Since it’s client-side storage, you can easily manage tokens through JavaScript without requiring additional HTTP requests.
3. Trade-offs Summary

Cookies:

#### Pros
Enhanced security with HttpOnly, Secure, and SameSite flags; simpler handling of cross-domain issues.

#### Cons
Client-side access is restricted, which can complicate some client-side logic that needs to interact with the token.

LocalStorage:

#### Pros
Easier implementation and better integration with client-side JavaScript; no additional HTTP overhead for JWT retrieval.

#### Cons
Lack of security features like HttpOnly and SameSite, making it more susceptible to XSS (Cross-Site Scripting) attacks. Also, the token can be exposed in browser developer tools.

#### Closing Summary

Storing JWTs in cookies versus localStorage involves a careful balance between security and usability. Cookies offer better protection against client-side exploits but come with additional complexity due to cross-domain limitations. LocalStorage, while simpler and more convenient for client-side logic, requires extra caution to prevent unauthorized access and potential security breaches.

#### Common Mistakes/Follow-Ups
Overlooking Security Flags: Failing to set the HttpOnly and Secure flags in cookies can leave the application vulnerable.
Not Considering SameSite Attribute: Not configuring the SameSite attribute properly could lead to unexpected behavior or security vulnerabilities.
Ignoring Client-Side Access Risks: Overlooking the risk of exposing tokens in client-side code through tools like browser developer consoles.

