---
type: "practice"
difficulty: "Basic"
---

# What does it mean to build a chatbot with memory in Python, and what is the simplest form of memory you could implement for a fresher project?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Token Budgeting for RAG](<../../../../TOOLS/Python/Python_Language/Token_Budgeting_for_RAG.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a chatbot with memory in Python means that our chatbot can retain information from previous interactions to provide more contextually relevant responses. For a beginner, implementing such functionality involves creating a simple form of memory where the bot remembers user inputs and uses this history to inform its current response.

#### Triage
The core of the question asks us to outline what it means to build a chatbot with memory in Python and then to provide an example of the simplest form of memory that could be implemented. This involves understanding how data can be stored and retrieved, which is crucial for maintaining context across multiple user interactions.

Key Assumptions:

We are working within the constraints of a basic chatbot project.
The solution should be straightforward yet functional to demonstrate understanding of core Python concepts.

#### First-principles Solution

Data Storage: To implement memory, we need a data structure that can store and retrieve information about past interactions. A dictionary is an excellent choice due to its key-value pair functionality.
User Input Tracking: We will associate each user's interaction with their unique identifier (like a username or session ID) in the dictionary.
Response Generation: The chatbot uses stored information from previous interactions to generate responses that are contextually relevant.

Here’s a step-by-step breakdown:

Initialize an empty dictionary to store conversation history.
Upon receiving each user input, check if it is a new user or returning by using their session ID as the key in the dictionary.
If it's a new user, add them to the dictionary. If it's a returning user, retrieve their previous interactions and use that context for the response.

A simple example:

PYTHON
# Initialize memory as an empty dictionary
memory = {}

def handle_user_input(session_id, input_text):
    # Check if this is a new session or a return
    if session_id not in memory:
        # If new user, initialize their history
        memory[session_id] = [input_text]
    else:
        # For returning users, add the current input to their history
        memory[session_id].append(input_text)

    # Example response generation using past inputs
    if len(memory[session_id]) <= 1:
        return "Welcome! Let's get started."
    else:
        last_input = memory[session_id][-2]  # Get the second to last input
        current_input = input_text  # Current user input
        return f"Based on your previous statement about {last_input}, you said: {current_input}"

# Example usage
print(handle_user_input("user123", "I love programming!"))  # Welcoming first response
print(handle_user_input("user123", "Python is my favorite language."))  # Using context from the previous input

#### Closing Summary
+ Common Mistakes/Follow-Ups: This simple memory implementation stores user interactions in a dictionary, allowing for contextual responses based on past conversations. Common mistakes could include not handling new users properly or forgetting to update the session history. Additional complexity could involve using more sophisticated data structures or integrating with external storage solutions.

**Code:**

```python
memory = {}
def handle_user_input(session_id, input_text):
    if session_id not in memory:
        memory[session_id] = [input_text]
    else:
        memory[session_id].append(input_text)

    if len(memory[session_id]) <= 1:
        return "Welcome! Let's get started."
    else:
        last_input = memory[session_id][-2]
        current_input = input_text
        return f"Based on your previous statement about {last_input}, you said: {current_input}"
```

**Example input / output:**

```text
Session 1:
- Input: "I love programming!"
Response: Welcome! Let's get started.

- Input: "Python is my favorite language."
Response: Based on your previous statement about I love programming!, you said: Python is my favorite language.
```

