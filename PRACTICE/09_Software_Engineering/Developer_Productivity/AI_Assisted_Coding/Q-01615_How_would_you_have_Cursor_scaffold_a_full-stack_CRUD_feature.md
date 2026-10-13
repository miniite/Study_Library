---
type: "practice"
difficulty: "Intermediate"
---

# How would you have Cursor scaffold a full-stack CRUD feature and what would you still write yourself?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Onboarding to Codebases with AI](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Onboarding_to_Codebases_with_AI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To scaffold a full-stack CRUD (Create, Read, Update, Delete) feature with AI-assisted coding, we'd leverage tools like GitHub Copilot to automatically generate code that complements our manual efforts. The goal is to focus on defining the high-level requirements and allowing the AI to handle much of the boilerplate code, while maintaining oversight over critical parts. Here’s a step-by-step approach:

Define High-Level Requirements: We need to outline what components we're working with (e.g., frontend framework, backend tech stack), the data model, endpoints, etc.
Choose Tools and Technologies: Decide on the technologies for front-end and back-end, such as React/Next.js for the front-end and Node.js with Express for the back-end.
Set Up Development Environment: Ensure all necessary tools are installed and configured correctly.

Now, let’s walk through how to use GitHub Copilot to scaffold this feature from first principles.

**Comparison:**

```
Operation	Endpoint	Method
Fetch all users	GET /api/users	GET
Create a new user	POST /api/users	POST
Update an existing user	PUT /api/users/:id	PUT
Delete a user	DELETE /api/users/:id	DELETE
```

**Code:**

```
MERMAID
Copy
graph TB;
    A[Define Project] --> B[Choose Technologies];
    B --> C[Set Up Development Environment];
    C --> D[Scaffold Backend API Using Express];
    D --> E[Scaffold Frontend Components];
```

**Example input / output:**

```
Backend (Express) Code Snippet
JAVASCRIPT
Copy
const express = require('express');
const app = express();
app.use(express.json());

// Mock data storage
let users = [];

// GET all users
app.get('/api/users', (req, res) => {
    res.status(200).json(users);
});

// POST a new user
app.post('/api/users', (req, res) => {
    const newUser = req.body;
    users.push(newUser);
    res.status(201).json(newUser);
});

// PUT update an existing user
app.put('/api/users/:id', (req, res) => {
    const id = req.params.id;
    const updatedUser = req.body;
    users = users.map(user => user.id === id ? updatedUser : user);
    res.status(200).json(updatedUser);
});

// DELETE a user
app.delete('/api/users/:id', (req, res) => {
    const id = req.params.id;
    users = users.filter(user => user.id !== id);
    res.status(204).send();
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
Frontend (React) Code Snippet
JAVASCRIPT
Copy
import React from 'react';
import { useState } from 'react';

function UserList() {
    const [users, setUsers] = useState([]);

    return (
        <div>
            <h1>User List</h1>
            <ul>
                {users.map(user => (
                    <li key={user.id}>{user.name}</li>
                ))}
            </ul>
        </div>
    );
}

function CreateUserForm() {
    const [name, setName] = useState('');

    const handleSubmit = (e) => {
        e.preventDefault();
        // Handle form submission
    };

    return (
        <form onSubmit={handleSubmit}>
            <input type="text" value={name} onChange={(e) => setName(e.target.value)} />
            <button type="submit">Create User</button>
        </form>
    );
}

function App() {
    return (
        <div>
            <UserList />
            <CreateUserForm />
        </div>
    );
}

export default App;
```

**Data:**

```text
- **Tech Stack**:
  - Frontend: React
  - Backend: Node.js with Express

- **Endpoints**:
  - `GET /api/users`: Fetch all users.
  - `POST /api/users`: Create a new user.
  - `PUT /api/users/:id`: Update an existing user.
  - `DELETE /api/users/:id`: Delete a user.
```

**Flow:**

1. Rendering diagram…

