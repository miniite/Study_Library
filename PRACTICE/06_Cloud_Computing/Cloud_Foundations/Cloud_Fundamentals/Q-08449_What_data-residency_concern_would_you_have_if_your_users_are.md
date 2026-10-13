---
type: "practice"
difficulty: "Intermediate"
---

# What data-residency concern would you have if your users are in India but your LLM provider processes prompts in the US?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Managed Services vs Self-hosting](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with AI models that process prompts and provide responses, a critical consideration is data residency, especially when user data originates from one country (India) while processing occurs in another (the US). For an AI engineer with intermediate experience, understanding the implications of this setup involves several key concerns. First, I would focus on compliance with regional data privacy laws, secondly, potential latency issues, and thirdly, ensuring secure transmission of data.

**Code:**

```text
# Example of a hypothetical data flow
def process_prompt(user_data):
    # Simulate data processing in the US
    processed_data = us_processing_center.process(user_data)
    return processed_data

def handle_user_request(user_input):
    # Simulate prompt from India user
    user_input = "What is the weather like today?"

    # Process the request
    result = process_prompt(user_input)
    print("Processed Result:", result)
```

**Example input / output:**

```

Input:

User input: What is the weather like today?

Output:

Processed result: (Assume the processed data returns a response)

Dry-run request-response:

CODE
Copy
User in India -> Sends prompt to US-based LLM provider
LLM Provider in US -> Processes and sends back response
Response received by user in India
```

**Data:**

```json
{
  "user_input": "What is the weather like today?",
  "processing_center_location": "US",
  "data_residency_required": "India"
}
```

