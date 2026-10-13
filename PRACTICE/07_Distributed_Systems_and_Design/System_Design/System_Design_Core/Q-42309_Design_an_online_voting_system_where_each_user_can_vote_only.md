---
type: "practice"
difficulty: "Intermediate"
---

# Design an online voting system where each user can vote only once.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Notification System](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Notification_System.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The core idea is to design a robust and scalable voting system that ensures each user votes only once. We'll use a combination of client-side validation, server-side checks, and database integrity to achieve this goal.

#### Triaging the Question

This question aims to assess how you would architect an online voting system with strict user validation rules. Specifically, we need to ensure that a user can vote only once per session or per unique identifier. The interviewer will be looking at your understanding of client-server interactions, state management, and database transactions.

#### First-Principles Solution

Client-Side Validation:

Use cookies or local storage on the client side to track if the user has voted.
Implement a simple check where the application prevents voting once a vote is recorded in these storages.

Server-Side Check and Database Integrity:

Use unique constraints on the database table storing votes.
During the voting process, the server checks for existing records with the same user identifier before allowing a new vote.

Session Management:

Implement session management to ensure that users cannot circumvent client-side validation by clearing cookies or using different browsers/sessions.

Scalability and Performance Considerations:

Ensure efficient database queries and minimal latency.
Use caching mechanisms (e.g., Redis) to handle high-concurrency scenarios without overwhelming the database.

Security Measures:

Protect against common security threats such as SQL injection, cross-site scripting (XSS), and man-in-the-middle attacks.
Detailed Steps:

Client-Side Validation with Cookies or Local Storage:

When a user attempts to vote, check if their session has already voted.
JAVASCRIPT
// Example JavaScript snippet for client-side validation
function castVote() {

#### cons
t cookieName = 'hasVoted';
    if (document.cookie.includes(cookieName)) {
        alert("You have already voted.");
        return;
    } else {
        document.cookie = `${cookieName}=true; path=/`;
        // Proceed with server request to vote
    }
}

Server-Side Implementation:

Create a database table to store votes, ensuring it has unique constraints.
SQL
CREATE TABLE Votes (
    id SERIAL PRIMARY KEY,
    voter_id VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Implement an API endpoint for voting that checks the voter_id:
PYTHON
from flask import Flask, request
import psycopg2

app = Flask(__name__)
conn = psycopg2.connect("dbname=voting_system user=postgres")

@app.route('/vote', methods=['POST'])
def vote():
    voter_id = request.form['voter_id']
    cursor = conn.cursor()
    try:
        cursor.execute("INSERT INTO Votes (voter_id) VALUES (%s)", (voter_id,))
        conn.commit()
        return "Vote cast successfully"
    except psycopg2.IntegrityError as e:
        conn.rollback()
        return "You have already voted", 400

Session Management:

Use session IDs or tokens to manage user sessions and prevent multiple votes from the same device.
JAVASCRIPT
// Example server-side session management using Flask
from flask import Flask, request, session

app = Flask(__name__)
app.secret_key = 'your_secret_key'

@app.route('/login', methods=['POST'])
def login():
    user_id = request.form['user_id']
    session['voter_id'] = user_id
    return "Logged in successfully"

@app.route('/vote')
def vote():
    voter_id = session.get('voter_id')
    if not voter_id:
        return "Login required to cast a vote"
    # Proceed with voting logic as above

Security Measures:

Implement HTTPS to secure the connection.
Use parameterized queries and prepared statements to prevent SQL injection.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Ensure you cover both client-side and server-side validations to prevent users from circumventing the system.
Discuss potential security vulnerabilities and how to mitigate them, such as using HTTPS, sanitizing input, and employing Content Security Policies (CSP).
Mention the importance of testing under high-concurrency scenarios to ensure the system remains responsive.

#### Common mistakes
include relying solely on client-side validation or not considering edge cases like distributed voting or device spoofing. Always consider the broader security landscape when designing such systems.

**Code:**

```

// Client-side cookie check for casting a vote function castVote() { const cookieName = 'hasVoted'; if (document.cookie.includes(cookieName)) { alert("You have already voted."); return; } else { document.cookie = ${cookieName}=true; path=/; // Proceed with server request to vote } }

CODE
Copy

```python
from flask import Flask, request
import psycopg2

app = Flask(__name__)
conn = psycopg2.connect("dbname=voting_system user=postgres")

@app.route('/vote', methods=['POST'])
def vote():
    voter_id = request.form['voter_id']
    cursor = conn.cursor()
    try:
        cursor.execute("INSERT INTO Votes (voter_id) VALUES (%s)", (voter_id,))
        conn.commit()
        return "Vote cast successfully"
    except psycopg2.IntegrityError as e:
        conn.rollback()
        return "You have already voted", 400
JAVASCRIPT
Copy
// Server-side session management using Flask
from flask import Flask, request, session

app = Flask(__name__)
app.secret_key = 'your_secret_key'

@app.route('/login', methods=['POST'])
def login():
    user_id = request.form['user_id']
    session['voter_id'] = user_id
    return "Logged in successfully"

@app.route('/vote')
def vote():
    voter_id = session.get('voter_id')
    if not voter_id:
        return "Login required to cast a vote"
    # Proceed with voting logic as above
```

**Data:**

```sql
-- Database schema for storing votes
CREATE TABLE Votes (
    id SERIAL PRIMARY KEY,
    voter_id VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Flow:**

1. No
2. Yes
3. No
4. User attempts to vote
5. Has user voted?
6. Generate unique voter_id
7. Is session authenticated?
8. Cast vote in database
9. Login required message
10. Vote successful message

