---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a REST endpoint that lets users give thumbs-up or thumbs-down feedback on an AI answer for later evaluation?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Filtering Sorting and Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Filtering_Sorting_and_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design a REST endpoint for allowing users to provide feedback (thumbs-up or thumbs-down) on AI-generated answers, we need to create a simple yet efficient API that handles both GET and POST requests. The endpoint should validate the input data and log the feedback in a database.

#### Triaging the Question

The question is asking us to design a REST API that allows users to provide feedback on an AI-generated answer. This involves understanding the requirements, designing the endpoint, ensuring security, and handling potential edge cases.

#### Assumptions
The system has a backend where the AI-generated answers are stored.
Each user can only submit one feedback per answer.
Feedback should be logged for future evaluation.
We need to handle HTTP methods (GET and POST).
Users will authenticate themselves before submitting feedback.

#### First-Principles Solution
Endpoint Design: Define the API endpoint, HTTP method, and URL path.
Input Validation: Ensure the input is valid before processing it.
Feedback Logging: Store user feedback in a database.
Security Considerations: Secure the endpoint to prevent unauthorized access.
Step-by-Step Solution

Define the Endpoint:

Endpoint: /api/answer/{answerId}/feedback
HTTP Method: POST
URL Path Parameters: {answerId}
Request Body: JSON object containing user ID and feedback type (thumbs-up or thumbs-down)

Input Validation:

Validate the answerId to ensure it exists in the database.
Ensure the request body contains valid data.
Check if the user has already provided feedback for this answer.

Feedback Logging:

Store the feedback in a database, logging the user ID, answer ID, and feedback type.

Security Considerations:

Use authentication to ensure only authenticated users can provide feedback.
Implement rate limiting to prevent spamming of feedback.

#### Closing Summary

To summarize, we have designed a REST API endpoint that allows users to submit feedback on AI-generated answers securely. The solution involves validating the input data, logging the feedback in a database, and ensuring security measures are in place to protect against unauthorized access and potential abuse.

#### Common Mistakes/Follow-Ups

Mistake: Not validating answerId or user ID before processing the request.

#### Follow-Up
Always validate all inputs and check for existing records before proceeding.

Mistake: Failing to log the feedback in a database.

#### Follow-Up
Ensure that the feedback is stored persistently.

Mistake: Lack of security measures, such as authentication and rate limiting.

#### Follow-Up
Implement robust security practices to protect against unauthorized access and spamming.

**Comparison:**

| User ID | Answer ID | Feedback Type |
|---------|-----------|---------------|
| 123456  | 789       | thumbs-up     |

**Code:**

```python
from flask import Flask, request, jsonify
import sqlite3

app = Flask(__name__)

# Database setup (simple example using SQLite)
def get_db_connection():
    conn = sqlite3.connect('feedback.db')
    return conn

@app.route('/api/answer/<int:answer_id>/feedback', methods=['POST'])
def provide_feedback(answer_id):
    # Get user ID and feedback type from request body
    data = request.json
    user_id = data.get('user_id')
    feedback_type = data.get('feedback_type')

    if not all([user_id, feedback_type]):
        return jsonify({"error": "Invalid input"}), 400

    # Validate answer ID and existence of the answer in the database
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM answers WHERE id=?", (answer_id,))
    result = cursor.fetchone()

    if not result:
        return jsonify({"error": "Answer not found"}), 404

    # Check if user has already provided feedback for this answer
    cursor.execute("SELECT * FROM feedbacks WHERE answer_id=? AND user_id=?", (answer_id, user_id))
    existing_feedback = cursor.fetchone()

    if existing_feedback:
        return jsonify({"error": "Feedback already exists"}), 409

    # Log the feedback in the database
    cursor.execute("INSERT INTO feedbacks (answer_id, user_id, feedback_type) VALUES (?, ?, ?)",
                   (answer_id, user_id, feedback_type))
    conn.commit()
    conn.close()

    return jsonify({"message": "Feedback submitted successfully"}), 201

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```
Request Body for POST /api/answer/{answerId}/feedback

{ "user_id": 123456, "feedback_type": "thumbs-up" }

CODE
Copy

```json
# Response on Success
HTTP/1.1 201 Created
Content-Type: application/json

{
    "message": "Feedback submitted successfully"
}

# Response on Failure - Missing Input
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "error": "Invalid input"
}

# Response on Answer Not Found
HTTP/1.1 404 Not Found
Content-Type: application/json

{
    "error": "Answer not found"
}

# Response on Duplicate Feedback
HTTP/1.1 409 Conflict
Content-Type: application/json

{
    "error": "Feedback already exists"
}
```

**Data:**

```sql
CREATE TABLE answers (
    id INTEGER PRIMARY KEY,
    content TEXT NOT NULL
);

CREATE TABLE feedbacks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    answer_id INTEGER NOT NULL,
    user_id INTEGER NOT NULL,
    feedback_type TEXT NOT NULL,
    FOREIGN KEY (answer_id) REFERENCES answers(id)
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Receive POST request
6. Is user authenticated?
7. Validate input and answer ID
8. Check if feedback exists forthis answer
9. Log feedback in database
10. Return 201 Created
11. Return 409 Conflict
12. Return 401 Unauthorized

