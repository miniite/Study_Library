---
type: "practice"
difficulty: "Basic"
---

# Why do multi-turn chatbots resend the whole conversation history on every REST request, and how does statelessness explain this?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Idempotent POST Requests](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Idempotent_POST_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Multi-turn chatbots often resend the entire conversation history with each REST request because they maintain a conversational context that needs to be consistent across multiple interactions. This behavior is closely tied to the principle of statelessness in REST APIs, which ensures that each API call can function independently without relying on external information.

To understand why this happens, let's first break down the core concepts and then delve into the solution step by step.

**Comparison:**

```
Step	Action	Output
1	Fetch conversation history.	["Hi there!", "Hello! How can I assist you today?", "Can you tell me about your services?", "Our services include AI development, chatbot implementation, and more."].
2	Process user message with context.	"Our services include AI development, chatbot implementation, and more." (if the user asks for details).
```

**Code:**

```python
# Example code snippet for a simple chatbot interaction (hypothetical)
class ChatBot:
    def handle_message(self, message):
        # Fetch conversation history from storage or session
        conversation_history = self.get_conversation_history()
        # Process the new message with existing context
        response = self.process_message(message, conversation_history)
        return response

    def get_conversation_history(self):
        # Simulate fetching conversation history (could be a database query)
        return ["User: Hi there!", "Bot: Hello! How can I assist you today?",
                "User: Can you tell me about your services?"]

    def process_message(self, message, context):
        # Process the message with current context
        response = f"Bot: {self.generate_response(message, context)}"
        return response

    def generate_response(self, message, context):
        # Generate a response based on the given message and context
        if "services" in message:
            return "Our services include AI development, chatbot implementation, and more."
        else:
            return "I'm not sure how to respond to that. Can you provide more details?"
```

**Example input / output:**

```text
Given a user message:
- User: Can you provide more details about your AI services?

The chatbot would have the following conversation history:
1. User: Hi there!
2. Bot: Hello! How can I assist you today?
3. User: Can you tell me about your services?
4. Bot: Our services include AI development, chatbot implementation, and more.

To process this message, the chatbot checks its stored conversation history to ensure context consistency before generating a response.
```

**Data:**

```json
{
  "conversation_history": [
    {"user": "Hi there!", "bot": "Hello! How can I assist you today?"},
    {"user": "Can you tell me about your services?", "bot": "Our services include AI development, chatbot implementation, and more."}
  ]
}
```

**Flow:**

1. Yes
2. No
3. User sends a new message
4. Is there conversation history?
5. Fetch conversation history
6. Process message with context
7. Generate response and sendback to user
8. Initialize empty context

