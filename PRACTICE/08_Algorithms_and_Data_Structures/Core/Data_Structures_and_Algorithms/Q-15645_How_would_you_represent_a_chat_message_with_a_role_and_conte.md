---
type: "practice"
difficulty: "Basic"
---

# How would you represent a chat message with a role and content, and why is a dictionary (or struct) a better fit than a plain tuple for the messages array an LLM API expects?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Tree Serialization](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tree_Serialization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To represent a chat message in Python, we need to encapsulate both the role of the sender and the content of the message. A common approach is to use either a dictionary or a struct-like object. Let's explore why a dictionary (or an equivalent struct) might be more suitable than a plain tuple for this purpose.

Firstly, let's define what each representation looks like:

Plain Tuple: A simple and lightweight option where we store the role and content as separate elements.

PYTHON
message = ("user", "Hello, how can I assist you today?")

Dictionary (or Struct-like Object): A more structured approach that allows for named fields, making it easier to access and manipulate the data.

PYTHON
message = {"role": "user", "content": "Hello, how can I assist you today?"}

When representing messages in an API expected by a Large Language Model (LLM), using a dictionary or struct has several advantages:

Named Fields: Dictionaries allow us to use named fields, making the code more readable and maintainable.
Flexibility: If we need to add more metadata about the message later (like timestamp or sender ID), it's straightforward in a dictionary but requires changes in a tuple.
Accessing Data: Accessing data is easier with dictionaries because you can use keys directly, e.g., message["role"] instead of message[0].

#### Cons
istency Across API: Many APIs expect messages to be structured as dictionaries or JSON objects, which makes it easier to integrate the chat message with the API.

Here's an example implementation using a dictionary:

PYTHON
def create_message(role, content):
    return {"role": role, "content": content}

message = create_message("user", "Hello, how can I assist you today?")
print(message)  # Output: {'role': 'user', 'content': 'Hello, how can I assist you today?'}

In contrast, a tuple would require additional logic to extract the values:

PYTHON
def extract_message_parts(tup):
    return tup[0], tup[1]

message = ("user", "Hello, how can I assist you today?")
role, content = extract_message_parts(message)
print(role)  # Output: user
print(content)  # Output: Hello, how can I assist you today?

In summary, using a dictionary or struct-like object (like the create_message function above) is generally more flexible and easier to work with compared to a plain tuple. It aligns better with common API structures and offers greater ease of access and modification.

**Code:**

```python
def create_message(role, content):
    return {"role": role, "content": content}

message = create_message("user", "Hello, how can I assist you today?")
print(message)  # Output: {'role': 'user', 'content': 'Hello, how can I assist you today?'}

def extract_message_parts(tup):
    return tup[0], tup[1]

message = ("user", "Hello, how can I assist you today?")
role, content = extract_message_parts(message)
print(role)  # Output: user
print(content)  # Output: Hello, how can I assist you today?
```

