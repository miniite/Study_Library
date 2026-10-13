---
type: "practice"
difficulty: "Intermediate"
---

# How would you use Copilot to write a function that stores conversation history for a chatbot, and what would you check about token growth over turns?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Verifying AI Generated ML Code](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Verifying_AI_Generated_ML_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an AI-assisted coding solution using Copilot for storing conversation history in a chatbot, we'll create a function that captures each message's metadata, such as user and bot messages, timestamps, etc. The key challenge is managing token growth efficiently to avoid hitting API limits or increasing costs. Here’s how we can approach this:

Firstly, the function will be responsible for collecting and storing each interaction in a structured format. We'll use JSON objects to represent each message with relevant metadata like sender (user or bot), timestamp, and content. The primary goal is to ensure that as the number of turns increases, our storage needs are managed effectively without compromising on performance.

**Code:**

```python
import json
from datetime import datetime

def store_conversation_history(user_message: str, bot_response: str) -> None:
    # Create a JSON object for the message with relevant metadata
    message = {
        "user_message": user_message,
        "bot_response": bot_response,
        "timestamp": datetime.now().isoformat(),
        "sender": "user"  # Placeholder, could be dynamic based on context
    }

    # Serialize and store the JSON object to a file or database
    with open("conversation_history.json", "a") as history_file:
        json.dump(message, history_file)
        history_file.write("\n")
```

