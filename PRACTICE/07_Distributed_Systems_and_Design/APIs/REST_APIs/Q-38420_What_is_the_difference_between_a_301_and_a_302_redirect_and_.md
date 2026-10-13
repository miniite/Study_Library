---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a 301 and a 302 redirect, and when might a REST API use them?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ REST Principles](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/REST_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In web development, understanding HTTP status codes is crucial, especially when dealing with redirects in REST APIs. The primary difference between a 301 (permanent) redirect and a 302 (temporary) redirect lies in their intended use cases and the impact on client caching behavior.

TL;DR: A 301 indicates that the resource has permanently moved to a new URL, while a 302 suggests that the request should be sent to a different URL temporarily. REST APIs use these redirects based on whether they expect the change in URL to be temporary or permanent for future requests.

#### Triage
Understanding the Question

The question is asking you to explain the distinction between 301 and 302 status codes, specifically within the context of a REST API. The interviewer might be probing your understanding of HTTP semantics, client caching behavior, and the appropriate use cases for each type of redirect.

#### Assumptions
You understand basic concepts of HTTP status codes.
You have experience with REST APIs and their design principles.

#### First Principles Solution
301 Redirect (Permanent)

A 301 redirect is used when a resource has moved to a new location permanently. This means that the original URL will not be recognized in the future, and any references or bookmarks should be updated to reflect the new URL.

Client Behavior: Search engines will update their index to point to the new URL.

#### Example
Use Case: If you are changing the domain of your application from oldapp.com to newapp.com, you would use a 301 redirect on all old URLs pointing to the corresponding resources in the new location.
302 Redirect (Temporary)

A 302 redirect is used when a resource has temporarily moved to another URL. This does not change the canonical URL of the resource, and search engines will typically treat it as if the user had navigated manually from one page to another.

Client Behavior: Search engines are less likely to update their index with the new location.

#### Example
Use Case: During a maintenance period or when performing updates that do not affect the long-term URL structure of your application, you might temporarily redirect users to a different page using 302.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, use 301 for permanent changes in URLs that should be reflected by search engines and clients. Use 302 for temporary changes or when the new URL is not intended as the canonical location.

#### Common mistakes
include using 302 for changes that are actually permanent, which can lead to issues with client caching and search engine indexing. Always consider the long-term implications of your redirect decisions.

#### Follow-ups
might involve discussing how these redirects affect cache invalidation or exploring alternative methods like URL rewriting in your application.

**Code:**

```
HTTP
Copy
# Example 301 Redirect Response
HTTP/1.1 301 Moved Permanently
Location: https://newapp.com/resource

# Example 302 Redirect Response
HTTP/1.1 302 Found
Location: http://maintenance.example.com
```

