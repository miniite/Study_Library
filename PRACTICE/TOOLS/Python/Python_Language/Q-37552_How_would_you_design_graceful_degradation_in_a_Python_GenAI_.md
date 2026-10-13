---
type: "practice"
difficulty: "Deep"
---

# How would you design graceful degradation in a Python GenAI service: when the primary model or vector store is down, what does the fallback return, and how do you make the degraded mode observable and bounded rather than silently wrong?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Retries with Deadline Budgets](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/Retries_with_Deadline_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Graceful degradation is crucial in ensuring a high-quality user experience when the primary components of an AI system are unavailable. In this scenario, we need to design a Python GenAI service that can handle the failure of the main model or vector store by providing fallback responses and maintaining observability. The key aspects include defining what the fallback response should be, how it differs from the main service, and ensuring bounded degradation without silently failing.

The question is essentially asking us to implement a robust fallback mechanism in Python. We need to consider the following:

Fallback Response: What kind of response should the system return when the primary model or vector store is down?
Observability: How can we make sure that the service's degraded state is observable and reported correctly?
Bounded Degradation: Ensuring that the fallback does not lead to unacceptable performance degradation.

We'll design a solution by first understanding the normal operation of our GenAI service, then defining the fallback behavior, and finally integrating observability mechanisms. Let's dive into each step in detail.

**Code:**

```python
import logging
from typing import Optional

# Mock classes to represent the primary model and vector store
class PrimaryModel:
    def __init__(self):
        self.is_down = False  # Simulate a model being down

    def predict(self, text: str) -> str:
        if not self.is_down:
            return f"Primary Model Prediction: {text}"
        else:
            raise Exception("Primary Model is Down")

class VectorStore:
    def __init__(self):
        self.is_down = False  # Simulate a vector store being down

    def get_vector(self, text: str) -> Optional[str]:
        if not self.is_down:
            return f"Vector for {text}"
        else:
            raise Exception("Vector Store is Down")

# Fallback response class
class FallbackResponseGenerator:
    def __init__(self, model: PrimaryModel, vector_store: VectorStore):
        self.model = model
        self.vector_store = vector_store

    def generate_response(self, text: str) -> str:
        try:
            primary_prediction = self.model.predict(text)
            return f"Final Prediction: {primary_prediction}"
        except Exception as e:
            logging.error(f"Primary service is down: {e}")
            fallback_response = self._fallback_generation(text)
            return fallback_response

    def _fallback_generation(self, text: str) -> str:
        # Basic fallback logic
        if self.vector_store.is_down:
            return f"Fallback Response (Vector Store Down): {text} - No vector available"
        else:
            fallback_vector = self.vector_store.get_vector(text)
            return f"Fallback Response (Model Down): {fallback_vector}"

# Example usage
if __name__ == "__main__":
    model = PrimaryModel()
    vector_store = VectorStore()

    # Simulate the primary service being down
    model.is_down = True

    fallback_generator = FallbackResponseGenerator(model, vector_store)
    response = fallback_generator.generate_response("Hello, how are you?")
    print(response)
```

**Example input / output:**

```plaintext
Fallback Response (Model Down): Vector for Hello, how are you?

This output demonstrates the fallback logic in action. When the primary model is down but the vector store is available, it uses the vector information as a fallback.
```

**Flow:**

1. Yes
2. No
3. Service Starts
4. Is Primary Model Down?
5. Fallback to Vector Store
6. Primary Model Predicts
7. Generate Fallback Responsewith Vector
8. Generate Prediction
9. Service Ends
10. This flowchart illustrates the decision-making process for the fallback mechanism.

