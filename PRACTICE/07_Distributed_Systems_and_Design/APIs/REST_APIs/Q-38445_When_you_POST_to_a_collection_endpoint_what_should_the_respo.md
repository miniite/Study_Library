---
type: "practice"
difficulty: "Basic"
---

# When you POST to a collection endpoint, what should the response body contain and why is returning the created resource often helpful?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ REST Principles](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/REST_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you POST to a collection endpoint in a REST API, the response body typically contains metadata about the newly created resource or the entire resource itself, depending on the requirements of the application and the context of the operation. Returning the created resource can be very helpful for several reasons:

Validation: If the request is successful, returning the full representation of the resource ensures that the client has immediate confirmation that the data was correctly processed.
Documentation: The response body provides clear documentation on what was actually created, which helps in understanding and debugging issues if they arise.
Client-side Actions: Clients can immediately update their local state or cache with the new data.

In essence, returning the created resource gives clients a complete picture of the outcome, enhancing the reliability and responsiveness of the application.

**Code:**

```

POST /api/users HTTP/1.1 Content-Type: application/json

{ "name": "John Doe", "email": "johndoe@example.com" }

CODE
Copy
HTTP/1.1 201 Created
Location: /api/users/5
Content-Type: application/json

{
  "id": 5,
  "name": "John Doe",
  "email": "johndoe@example.com",
  "createdAt": "2023-10-04T14:48:00Z"
}
```

**Example input / output:**

```text
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "Jane Doe",
  HTTP/1.1 201 Created
Location: /api/users/6
Content-Type: application/json

{
  "id": 6,
  "name": "Jane Doe",
  "email": "janedoe@example.com",
  "createdAt": "2023-10-04T15:08:00Z"
}
```

