---
type: "practice"
difficulty: "Intermediate"
---

# How would you keep a running conversation with an LLM in Python by appending each turn to a messages list, and what problem grows as the conversation gets long?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming LLM Responses](<../../../../TOOLS/Python/Python_Language/Streaming_LLM_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To maintain a running conversation with a Language Model (LLM) in Python, you can append each user- or model-generated turn of dialogue into a messages list. This approach is straightforward and effective for short conversations but poses challenges as the conversation lengthens.

#### Triaging the Question

The question asks about maintaining a conversation context using an LLM via appending messages to a list. The core problem is managing memory usage and ensuring that the context remains relevant over many turns, while also handling any potential limitations in API request lengths or model input limits.

#### First-Principles Solution
Define the messages List: Initialize a list where each element represents a turn in the conversation.
Context Management:
For each new user input, append it to the messages list and pass this context along with any relevant previous turns to the LLM for generating the next response.
Ensure that the model’s input length is managed so as not to exceed API limits.
Conversation Context Limitation: As conversations grow longer, manage the context by either:
Trimming older messages if they are no longer critical or
Splitting the conversation into smaller chunks for processing.
Code Example

Below is a Python implementation that manages this conversation flow:

PYTHON
class ConversationManager:
    def __init__(self, max_context_length=10):
        self.messages = []
        self.max_context_length = max_context_length

    def add_user_message(self, message):
        # Append user message to the list and manage context length
        self.messages.append({"role": "user", "content": message})
        if len(self.messages) > self.max_context_length:
            self.messages.pop(0)

    def get_response_from_llm(self, model, messages):
        # Placeholder for actual LLM API call; this is where you would integrate your LLM
        response = self.generate_dummy_response()
        return response

    def generate_dummy_response(self):
        # Dummy function to simulate an LLM response
        if len(self.messages) == 0:
            return "Hello! How can I assist you today?"
        else:
            last_message = self.messages[-1]["content"]
            return f"Understood, {last_message}. What's next?"

# Example usage
manager = ConversationManager()
manager.add_user_message("What is the weather like today?")
print(manager.get_response_from_llm(None, manager.messages))

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: The key idea is to manage a running conversation context by appending messages and ensuring that the context length remains within acceptable limits.

#### Common Mistakes
Failing to manage memory usage as conversations grow long, leading to performance issues or API request failures.
Not considering edge cases where users might input very large messages, which could exceed allowed character lengths.
Further Considerations
Efficiency: Ensure that the context management logic is efficient, especially if dealing with frequent user interactions.
Scalability: Consider how this approach scales with more complex or longer conversations, possibly integrating techniques like summarization to keep the context relevant without losing important information.

