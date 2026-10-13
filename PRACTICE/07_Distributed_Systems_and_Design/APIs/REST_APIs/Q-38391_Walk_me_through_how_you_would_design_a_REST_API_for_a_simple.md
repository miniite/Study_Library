---
type: "practice"
difficulty: "Basic"
---

# Walk me through how you would design a REST API for a simple to-do list application.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Filtering Sorting and Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Filtering_Sorting_and_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a REST API for a simple to-do list application, we need to consider several key aspects such as resource management, HTTP methods, and data formats. The core idea is to ensure that our API is easy to use, scalable, and secure while maintaining consistency with REST principles.

#### Triage

The question asks us to design a REST API for a to-do list application.
We need to define resources (e.g., tasks), operations on these resources, and the HTTP methods used to interact with them.
The interviewer is probing our understanding of RESTful design patterns, HTTP verbs, and data formats.

#### First-principles solution

Define Resources: In this case, we have a single resource: tasks.
HTTP Methods:
GET /tasks: Fetch all tasks or fetch specific tasks based on filters.
POST /tasks: Create a new task.
PUT /tasks/{id}: Update an existing task.
DELETE /tasks/{id}: Delete a task.
Data Formats:
Use JSON for request and response bodies.

Here’s a detailed breakdown:

GET /tasks
Description: Fetch all tasks or specific tasks based on filters.

#### Example
Request:
HTTP
GET /tasks?completed=true
Response Example (JSON):
JSON
{
  "tasks": [
    {
      "id": 1,
      "title": "Buy groceries",
      "description": "Get milk, eggs, and bread.",
      "completed": false,
      "created_at": "2023-10-05T14:30:00Z"
    }
  ]
}
POST /tasks
Description: Create a new task.

#### Example
Request:
HTTP
POST /tasks
Content-Type: application/json

{
  "title": "Write code",
  "description": "Finish the user authentication module.",
  "completed": false
}
Response Example (JSON):
JSON
{
  "id": 2,
  "title": "Write code",
  "description": "Finish the user authentication module.",
  "completed": false,
  "created_at": "2023-10-05T14:35:00Z"
}
PUT /tasks/{id}
Description: Update an existing task.

#### Example
Request:
HTTP
PUT /tasks/1
Content-Type: application/json

{
  "title": "Write code",
  "description": "Fix the bug in the user authentication module.",
  "completed": true
}
Response Example (JSON):
JSON
{
  "id": 1,
  "title": "Write code",
  "description": "Fix the bug in the user authentication module.",
  "completed": true,
  "updated_at": "2023-10-05T14:40:00Z"
}
DELETE /tasks/{id}
Description: Delete a task.

#### Example
Request:
HTTP
DELETE /tasks/1
Response Example (JSON):
JSON
{
  "status": "success",
  "message": "Task deleted successfully."
}
Summary and Common Mistakes/Follow-Ups:
Ensure all CRUD operations are covered.
Use appropriate HTTP verbs for each action.
Validate input data to prevent errors.
Handle edge cases, such as non-existent tasks or invalid requests.

**Comparison:**

```
Operation	HTTP Method	Path	Description
Fetch all tasks or specific tasks	GET	/tasks?completed=true	Fetch all tasks or specific tasks based on filters.
Create a new task	POST	/tasks	Create a new task.
Update an existing task	PUT	/tasks/{id}	Update an existing task.
Delete a task	DELETE	/tasks/{id}	Delete a task.
```

**Code:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

tasks = []

@app.route('/tasks', methods=['GET'])
def get_tasks():
    return jsonify(tasks)

@app.route('/tasks', methods=['POST'])
def create_task():
    new_task = request.json
    tasks.append(new_task)
    return jsonify(new_task), 201

@app.route('/tasks/<int:task_id>', methods=['PUT'])
def update_task(task_id):
    task = next((t for t in tasks if t['id'] == task_id), None)
    if not task:
        return jsonify({"error": "Task not found"}), 404
    updated_data = request.json
    task.update(updated_data)
    return jsonify(task)

@app.route('/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    global tasks
    tasks = [t for t in tasks if t['id'] != task_id]
    return jsonify({"status": "success", "message": "Task deleted successfully."})

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```
GET /tasks?completed=true

GET /tasks?completed=true

HTTP/1.1 200 OK Content-Type: application/json

{ "tasks": [ { "id": 1, "title": "Buy groceries", "description": "Get milk, eggs, and bread.", "completed": false, "created_at": "2023-10-05T14:30:00Z" } ] }

CODE
Copy

```http
# POST /tasks
POST /tasks

Content-Type: application/json

{
  "title": "Write code",
  "description": "Finish the user authentication module.",
  "completed": false
}

HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 3,
  "title": "Write code",
  "description": "Finish the user authentication module.",
  "completed": false,
  "created_at": "2023-10-05T14:37:00Z"
}
HTTP
Copy
# PUT /tasks/1
PUT /tasks/1

Content-Type: application/json

{
  "title": "Write code",
  "description": "Fix the bug in the user authentication module.",
  "completed": true
}

HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "title": "Write code",
  "description": "Fix the bug in the user authentication module.",
  "completed": true,
  "updated_at": "2023-10-05T14:40:00Z"
}
HTTP
Copy
# DELETE /tasks/1
DELETE /tasks/1

HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "success",
  "message": "Task deleted successfully."
}
```

**Data:**

```json
{
  "tasks": [
    {
      "id": 1,
      "title": "Buy groceries",
      "description": "Get milk, eggs, and bread.",
      "completed": false,
      "created_at": "2023-10-05T14:30:00Z"
    },
    {
      "id": 2,
      "title": "Write code",
      "description": "Finish the user authentication module.",
      "completed": false,
      "created_at": "2023-10-05T14:35:00Z"
    }
  ]
}
```

**Flow:**

1. true
2. false
3. yes
4. no
5. yes
6. no
7. yes
8. no
9. GET /tasks
10. Completed?
11. Filter tasks
12. Return all tasks
13. POST /tasks
14. Create new task and return ID
15. PUT /tasks/{id}
16. Task exists?
17. Update task details and returnupdated data
18. Error message or status code?
19. Return error message
20. DELETE /tasks/{id}
21. Task exists?
22. Delete task and return successmessage

