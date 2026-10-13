---
type: "practice"
difficulty: "Intermediate"
---

# How would you design REST endpoints to create a conversation, add messages to it, and fetch its history for a chat application?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Filtering Sorting and Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Filtering_Sorting_and_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design REST endpoints for a chat application involving conversations, messages, and fetching their history, we need to focus on clear, intuitive URLs, HTTP methods that match CRUD (Create, Read, Update, Delete) operations, and proper error handling. Here’s how we can break down the problem:

TL;DR: We'll design REST endpoints for creating a conversation, adding messages to it, and fetching its history. Each endpoint will use appropriate HTTP verbs and return meaningful responses.

#### Triage
The question is essentially asking us to define API endpoints that enable users to interact with conversations in a chat application. The interviewer may be probing our understanding of REST principles, error handling, and the choice of HTTP methods.

#### First-Principles Solution

Create Conversation: Use POST /conversations to create a new conversation.
Add Messages to Conversation: Use POST /conversations/{conversation_id}/messages to add messages to an existing conversation.
Fetch Conversation History: Use GET /conversations/{conversation_id} to fetch the history of a specific conversation.

Let’s go through each step in detail:

1. Create Conversation
Endpoint: POST /conversations
Request Body:
JSON
{
  "title": "Interview Prep",
  "participants": ["user1@example.com", "user2@example.com"]
}
Response on Success:
JSON
{
  "id": "1234567890abcdef",
  "title": "Interview Prep",
  "participants": ["user1@example.com", "user2@example.com"],
  "messages": []
}
2. Add Messages to Conversation
Endpoint: POST /conversations/{conversation_id}/messages
Request Body:
JSON
{
  "text": "Hello, how's the interview going?"
}
Response on Success:
JSON
{
  "id": "1234567890abcdef",
  "conversation_id": "9876543210abcde",
  "text": "Hello, how's the interview going?"
}
3. Fetch Conversation History
Endpoint: GET /conversations/{conversation_id}
Response on Success:
JSON
{
  "id": "9876543210abcde",
  "title": "Interview Prep",
  "participants": ["user1@example.com", "user2@example.com"],
  "messages": [
    {
      "id": "1234567890abcdef",
      "text": "Hello, how's the interview going?",
      "sender": "user1@example.com",
      "timestamp": "2023-10-05T14:00:00Z"
    }
  ]
}

#### Assumptions
Each user has a unique identifier.
Conversations can have multiple messages.
Messages include the sender and timestamp.

#### Common Mistakes/Follow-Ups
Mistake: Using GET for creating or updating resources, which is not idempotent.

#### Follow-Up
Discussing the importance of error handling (e.g., 404 Not Found if conversation does not exist).

**Comparison:**

```
Endpoint	Method	Description	HTTP Status Codes
/conversations	POST	Create a new conversation.	201 Created, 409 Conflict (if title is already in use)
/conversations/{conversation_id}/messages	POST	Add a message to an existing conversation.	201 Created, 404 Not Found (if conversation does not exist)
/conversations/{conversation_id}	GET	Fetch the history of a specific conversation.	200 OK, 404 Not Found (if conversation does not exist)
```

**Code:**

```python
# Example code snippet for adding a message to a conversation
from flask import Flask, request, jsonify

app = Flask(__name__)

conversations = {}

@app.route('/conversations', methods=['POST'])
def create_conversation():
    data = request.get_json()
    new_conversation_id = len(conversations) + 1
    conversations[new_conversation_id] = {
        "title": data['title'],
        "participants": data['participants'],
        "messages": []
    }
    return jsonify({
        "id": new_conversation_id,
        "title": data['title'],
        "participants": data['participants'],
        "messages": []
    }), 201

@app.route('/conversations/<int:conversation_id>/messages', methods=['POST'])
def add_message(conversation_id):
    if conversation_id not in conversations:
        return jsonify({"error": "Conversation not found"}), 404
    data = request.get_json()
    new_message_id = len(conversations[conversation_id]['messages']) + 1
    conversations[conversation_id]['messages'].append({
        "id": new_message_id,
        "text": data['text'],
        "sender": "user@example.com",
        "timestamp": "2023-10-05T14:00:00Z"
    })
    return jsonify({"id": new_message_id, "conversation_id": conversation_id, "text": data['text']})

@app.route('/conversations/<int:conversation_id>', methods=['GET'])
def fetch_conversation(conversation_id):
    if conversation_id not in conversations:
        return jsonify({"error": "Conversation not found"}), 404
    return jsonify({
        "id": conversation_id,
        "title": conversations[conversation_id]['title'],
        "participants": conversations[conversation_id]['participants'],
        "messages": conversations[conversation_id]['messages']
    })
```

**Example input / output:**

| **Input** | **Output (Success)** |
|-----------|----------------------|
| POST /conversations | {"id": 1, "title": "Interview Prep", "participants": ["user1@example.com", "user2@example.com"], "messages": []} |
| POST /conversations/1/messages | {"id": 1, "conversation_id": 1, "text": "Hello, how's the interview going?"} |
| GET /conversations/1 | {"id": 1, "title": "Interview Prep", "participants": ["user1@example.com", "user2@example.com"], "messages": [{"id": 1, "text": "Hello, how's the interview going?", "sender": "user@example.com", "timestamp": "2023-10-05T14:00:00Z"}]} |

**Data:**

```json
{
  "title": "Interview Prep",
  "participants": ["user1@example.com", "user2@example.com"]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. POST /conversations
8. Conversation Exists?
9. Create New Conversation
10. Error: Duplicate Title
11. Return Created Conversation
12. Conversation ID Valid?
13. POST/conversations/{conversation_id}/messages
14. Error: Invalid Conversation ID
15. Message Exists?
16. Create New Message
17. Error: Invalid Request
18. Return Created Message
19. Return Error: InvalidConversation ID

