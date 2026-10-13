---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks you to design a 'you may also like' recommendation module at class level with a fallback when the ML service is down; where do you draw the interface so the fallback is a swappable strategy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Threaded Comments Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Threaded_Comments_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this low-level design (LLD) problem, we need to break it down into several key components and interfaces that ensure flexibility and robustness in our recommendation system. The core challenge is designing a fallback mechanism that can seamlessly take over when the machine learning (ML) service is unavailable.

#### Triaging the Question

The question asks us to design an interface for a 'you may also like' recommendation module with a swappable fallback strategy. This involves understanding how the ML model and the fallback mechanism interact, ensuring both are independently testable and maintainable. The key aspects we need to consider include:

Interface Design: Where do we draw the line between the recommendation engine and other components?
Fallback Mechanism: How can we ensure that the system can gracefully degrade if the ML service is down without compromising user experience?
Scalability and Maintainability: What are the trade-offs in terms of performance and complexity?

#### First Principles Solution

Identify Components:

ML Model Service: The primary recommendation engine powered by a trained ML model.
Fallback Mechanism: A secondary strategy that can be swapped out easily to provide recommendations when the ML service is unavailable.

Interface Design:

Recommendation Engine Interface: This interface will define how the main application interacts with the recommendation system. It should include methods for fetching user preferences, historical data, and generating personalized recommendations.
Fallback Strategy Interface: Define a common interface that any fallback mechanism must adhere to. This ensures that the main application can seamlessly switch between different strategies.

#### Implementation

Swappable Fallback Mechanism: Implement the fallback strategy as an independent module that implements the same interface as the ML model service.
Conditional Execution: Use conditional logic in the main recommendation flow to check the availability of the ML service and delegate execution based on this condition.

#### Example
Code:

PYTHON
from typing import Any, Callable

class RecommendationStrategyInterface:
    def get_recommendations(self, user_id: int) -> list[Any]:
        pass

class MlRecommendationService(RecommendationStrategyInterface):
    def __init__(self, model: object):
        self.model = model

    def get_recommendations(self, user_id: int) -> list[Any]:
        # Code to fetch recommendations using the ML model
        return self.model.predict(user_id)

class FallbackRecommendationService(RecommendationStrategyInterface):
    def __init__(self, strategy: Callable[[int], list[Any]]):
        self.strategy = strategy

    def get_recommendations(self, user_id: int) -> list[Any]:
        # Code to provide fallback recommendations
        return self.strategy(user_id)

class RecommendationService:
    def __init__(self, ml_service: MlRecommendationService,
                 fallback_service: FallbackRecommendationService):
        self.ml_service = ml_service
        self.fallback_service = fallback_service

    def get_recommendations(self, user_id: int) -> list[Any]:
        if is_ml_service_available():
            return self.ml_service.get_recommendations(user_id)
        else:
            return self.fallback_service.get_recommendations(user_id)

def is_ml_service_available() -> bool:
    # Check the availability of the ML service
    return True  # Placeholder for actual check

# Example usage
ml_service = MlRecommendationService(model)
fallback_service = FallbackRecommendationService(lambda uid: [default_recommendation()])
recommendation_service = RecommendationService(ml_service, fallback_service)

user_id = 12345
recommendations = recommendation_service.get_recommendations(user_id)

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By defining clear interfaces for the recommendation engine and a swappable fallback strategy, we ensure that our system is robust and maintainable. The fallback mechanism can be easily swapped out or extended without affecting the main application logic.

#### Common Mistakes
Not defining an explicit interface: This can lead to tight coupling between the components, making maintenance difficult.
Ignoring performance implications of the fallback strategy: A naive fallback might degrade user experience if it is too slow or resource-intensive.

#### Follow-Ups
Discuss additional considerations such as caching strategies, fallback priority, and how to monitor the health of both the ML service and the fallback mechanism.

**Code:**

```python
from typing import Any

class RecommendationStrategyInterface:
    def get_recommendations(self, user_id: int) -> list[Any]:
        pass

class MlRecommendationService(RecommendationStrategyInterface):
    def __init__(self, model: object):
        self.model = model

    def get_recommendations(self, user_id: int) -> list[Any]:
        # Code to fetch recommendations using the ML model
        return self.model.predict(user_id)

class FallbackRecommendationService(RecommendationStrategyInterface):
    def __init__(self, strategy: Callable[[int], list[Any]]):
        self.strategy = strategy

    def get_recommendations(self, user_id: int) -> list[Any]:
        # Code to provide fallback recommendations
        return self.strategy(user_id)

class RecommendationService:
    def __init__(self, ml_service: MlRecommendationService,
                 fallback_service: FallbackRecommendationService):
        self.ml_service = ml_service
        self.fallback_service = fallback_service

    def get_recommendations(self, user_id: int) -> list[Any]:
        if is_ml_service_available():
            return self.ml_service.get_recommendations(user_id)
        else:
            return self.fallback_service.get_recommendations(user_id)

def is_ml_service_available() -> bool:
    # Check the availability of the ML service
    return True  # Placeholder for actual check

# Example usage
ml_service = MlRecommendationService(model)
fallback_service = FallbackRecommendationService(lambda uid: [default_recommendation()])
recommendation_service = RecommendationService(ml_service, fallback_service)

user_id = 12345
recommendations = recommendation_service.get_recommendations(user_id)
```

