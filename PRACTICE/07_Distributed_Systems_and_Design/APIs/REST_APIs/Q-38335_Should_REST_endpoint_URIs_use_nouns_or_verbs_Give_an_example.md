---
type: "practice"
difficulty: "Intermediate"
---

# Should REST endpoint URIs use nouns or verbs? Give an example of good and bad naming.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ REST Principles](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/REST_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a RESTful API, one key decision is whether to name your endpoints using nouns (resources) or verbs (actions). Nouns are generally preferred because they clearly identify what data you're working with, making the API more intuitive and self-documenting. Verbs can be useful but should be used sparingly as they might complicate the URI structure.

Let's explore both scenarios with examples to understand the nuances better.

**Code:**

```plaintext
# Example of a good noun-based naming in REST API

GET /users        # Retrieve a list of users
GET /users/{id}   # Retrieve a specific user by ID
POST /users       # Create a new user
PUT /users/{id}   # Update an existing user
DELETE /users/{id}# Delete a user

# Example of a bad verb-based naming in REST API

POST /user/verify  # Verify the user (bad, as it's not clear if this is creating or updating the verification)
PUT /user/login    # Log in the user (verb "login" suggests an action rather than a resource)
DELETE /users/auth # Unauthorize access (imprecise and unclear)
```

