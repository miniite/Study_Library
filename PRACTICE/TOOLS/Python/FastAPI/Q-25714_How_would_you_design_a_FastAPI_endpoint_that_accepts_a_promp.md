---
type: "practice"
difficulty: "Deep"
---

# How would you design a FastAPI endpoint that accepts a prompt-template version id and routes to the corresponding template so you can A/B two prompt variants with sticky user bucketing?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Batching LLM Requests](<../../../../TOOLS/Python/FastAPI/Batching_LLM_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an A/B testing endpoint in FastAPI that routes to the correct prompt template based on a provided version ID, while implementing sticky user bucketing, we need to ensure that each user is consistently routed to one of two variants throughout their session. This involves understanding how to handle routing logic and maintaining state across requests for each user.

First, let's clarify what the question is asking:

The endpoint should accept a version_id parameter.
Two versions of prompt templates need to be A/B tested.
Users should be routed to one of these two variants in a sticky manner, meaning they should remain on that variant for their entire session.

#### Assumptions

Each user has a unique identifier (e.g., session ID or username) that can be used to track which variant they are currently assigned to.
The version_id parameter is used to identify the specific template being requested.

#### First Principles Solution
Define Endpoint: Create an endpoint in FastAPI that accepts a query parameter for the version_id.
User State Management: Use session or user state management to keep track of which variant each user is currently assigned.
Routing Logic: Implement logic to route users to one of the two variants based on their current state.
Step-by-Step Solution
Define Endpoint and Data Models:
Create a FastAPI endpoint that accepts version_id as a query parameter.
Use session or in-memory storage to manage user states (variant assignment).
PYTHON
from fastapi import FastAPI, Query
from typing import Dict

app = FastAPI()

# Simulated user state management using an in-memory dictionary
user_state: Dict[str, int] = {}

@app.get("/prompt_template")
async def get_prompt_template(version_id: str = Query(...)):
    # Fetch the current variant assignment for the user or create a new entry if not present
    session_id = "user_session_id"  # Replace with actual session ID logic
    if session_id in user_state:
        assigned_variant = user_state[session_id]
    else:
        # Randomly assign to one of two variants on first request
        assigned_variant = int(version_id) % 2
        user_state[session_id] = assigned_variant

    return {"variant": assigned_variant, "version_id": version_id}

Routing Logic:

The logic inside the endpoint ensures that each user is always routed to the same variant for their session.
This can be achieved by using a simple modulo operation on version_id (since we have only two variants).

Handling Edge Cases and Failures:

Ensure that users are not repeatedly reassigned if they request the endpoint multiple times within their session.
Handle cases where version_id is missing or invalid.

Sticky User Routing:

Use a mechanism to store user state (e.g., in-memory dict, Redis) so that routing decisions persist across requests for each user.

#### Closing Summary

In summary, the solution involves creating an endpoint with FastAPI that uses session management to ensure sticky routing of users between two variants. By leveraging a simple modulo operation on version_id, we can achieve consistent variant assignment per user throughout their session.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to properly manage user state, leading to inconsistent routing.

#### Follow-Up
Discussing the scalability of in-memory storage versus using a database or cache (e.g., Redis) for storing user states.
Mistake: Not handling edge cases where version_id is missing or invalid.

#### Follow-Up
Adding validation and error handling logic to ensure robustness.

