---
type: "practice"
difficulty: "Basic"
---

# How is the API key sent when calling the Anthropic API, and why is it placed in a header rather than in the URL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ API Keys vs Tokens](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Keys_vs_Tokens.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When making requests to the Anthropic API, an API key should be included as part of the request headers rather than being embedded directly in the URL. This approach ensures better security and flexibility compared to placing it in the query string.

To summarize:

API Key Placement: The API key is sent within the Authorization header.
Security: Placing sensitive information like API keys in HTTP headers makes them more secure because they are not visible in the URL, which can be logged by various services (e.g., web servers, proxies).
Flexibility and Standardization: Using standard HTTP headers such as Authorization is a common practice that aligns with RESTful API design principles.

#### Triage

The question asks about how to send an API key when making requests to the Anthropic API and why it's placed in a header rather than in the URL. The core idea here is understanding the security and design considerations behind HTTP headers versus query strings for transmitting sensitive information like API keys.

#### First-Principles Solution

Let’s break down the steps from first principles:

HTTP Headers vs Query Strings:

Headers (like Authorization): These are metadata that provide context about the request and can include sensitive data.
Query Strings: This is part of the URL and can be logged by various services, making it less secure for transmitting API keys.

Security Considerations:

When you send an API key in a query string (e.g., https://api.anthropic.com/v1/complete?api-key=12345), the key is visible to anyone who logs network requests or intercepts the traffic. This poses significant security risks.
Placing the API key in the Authorization header (Authorization: Bearer 12345) ensures that it remains hidden from view, enhancing security.

Standard Practice:

The Authorization header is commonly used for sending authentication credentials securely. It’s a standard practice across many APIs and protocols like OAuth.
For example, in the case of Anthropic API (or similar APIs), you would set up your request as follows:
HTTP
GET /v1/complete HTTP/1.1
Host: api.anthropic.com
Authorization: Bearer YOUR_API_KEY_HERE
API Documentation:
Always refer to the official API documentation for specific instructions. For Anthropic, their API docs will clearly state that you should use an Authorization header.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, sending an API key via a standard HTTP header like Authorization is crucial for maintaining security and adhering to best practices in RESTful API design. It’s important not to expose sensitive information in the URL or query string.

#### Common Mistakes

Forgetting to include necessary headers.
Incorrectly formatting the Authorization header (e.g., missing Bearer  prefix).
Not checking the official API documentation for specific requirements and supported methods.

For follow-ups, you might be asked about other authentication mechanisms or how to handle rate limiting and authorization in your requests. Always ensure that any sensitive information is handled securely, adhering to best practices.

**Code:**

```
HTTP
Copy
GET /v1/complete HTTP/1.1
Host: api.anthropic.com
Authorization: Bearer YOUR_API_KEY_HERE
```

