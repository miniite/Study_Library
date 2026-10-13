---
type: "practice"
difficulty: "Deep"
---

# Design the model-routing and fallback layer for a product that uses five different LLMs by cost and capability — how do you route per-request degrade to a cheaper model on overload and keep the abstraction stable as providers deprecate models every quarter?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Backpressure and Load Shedding](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Backpressure_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust model-routing and fallback layer, we need to ensure efficient handling of requests across multiple Language Models (LLMs) based on their cost and capabilities while maintaining abstraction stability even as providers deprecate models quarterly. The key steps involve understanding the LLMs' costs and capabilities, defining a routing strategy, implementing fallback mechanisms, and ensuring that the system can adapt over time.

Firstly, let’s consider the core components of our design:

Model Registry: A centralized repository to keep track of available LLMs, their costs, and capabilities.
Routing Logic: Determining which model to use for a request based on cost and capability requirements.
Fallback Mechanism: Handling overloads by degrading gracefully to cheaper models.
Abstraction Layer: Ensuring that changes in the underlying model implementations do not affect the application’s logic.

Let's break this down step-by-step:

#### Triaging the Question

The primary goals are:

Efficient routing of requests based on cost and capability.
Graceful degradation during overloads by switching to cheaper models.
Keeping the abstraction layer stable despite frequent updates in provider offerings.

Key assumptions include:

The LLMs have varying capabilities and costs, which need to be managed.
Providers deprecate models every quarter, requiring dynamic updates.
The system must handle load effectively without losing functionality.

#### First Principles Solution
Model Registry: Create a model registry that maps each LLM by its unique identifier (e.g., model_id), cost (cost_per_request), and capabilities (e.g., max tokens, language support).
Routing Logic:
Define a scoring function that prioritizes models based on their cost and capability.
Implement a load-balancing mechanism to distribute requests efficiently across the available models.
Fallback Mechanism:
Design a fallback strategy where higher-cost models are chosen first, but if a model is unavailable or overloading, switch to cheaper alternatives.
Abstraction Layer:
Ensure that the application logic remains decoupled from the underlying LLMs by using interfaces and adapters.
Detailed Implementation

Model Registry

PYTHON
class ModelRegistry:
    def __init__(self):
        self.models = {
            "model1": {"cost_per_request": 0.5, "max_tokens": 2048},
            "model2": {"cost_per_request": 0.75, "max_tokens": 1024},
            # Add more models
        }

Routing Logic

PYTHON
class ModelRouter:
    def __init__(self, registry):
        self.registry = registry

    def route_request(self, request):
        available_models = sorted(
            [(model_id, model) for model_id, model in self.registry.models.items()],
            key=lambda x: (x[1]["cost_per_request"], -x[1]["max_tokens"]),
        )
        for model_id, _ in available_models:
            if self.check_model_availability(model_id):
                return model_id

Fallback Mechanism

PYTHON
class FallbackManager:
    def handle_overload(self, model_router, request):
        while True:
            selected_model = model_router.route_request(request)
            try:
                # Attempt to use the selected model
                response = self.execute_model(selected_model, request)
                return response
            except ModelUnavailableError:
                pass  # Fallback to cheaper models

    def execute_model(self, model_id, request):
        if not self.check_model_availability(model_id):
            raise ModelUnavailableError("Model is unavailable")
        # Execute the selected model

Abstraction Layer

PYTHON
class LLMAdapter:
    def __init__(self, model_id):
        self.model = self.load_model(model_id)

    def load_model(self, model_id):
        return ModelProvider().get_model_by_id(model_id)  # Interface to the provider

    def generate_response(self, request):
        response = self.model.generate(request)
        return response

#### Closing Summary
and Common Mistakes/Follow-Ups

This design ensures that requests are efficiently routed based on cost and capability while providing robust fallback mechanisms during overloads. The abstraction layer keeps the application logic stable as models change.

#### Common mistakes
include:

Not defining clear criteria for model routing.
Failing to implement proper fallback strategies, leading to service disruptions.
Overcomplicating the design without considering real-world constraints like provider deprecation cycles.

#### Follow-ups
might involve discussing specific edge cases, such as handling requests during model migrations or integrating with monitoring tools to track load and performance.

**Comparison:**

```
Model ID	Cost Per Request ($)	Max Tokens
model1	0.5	2048
model2	0.75	1024
```

**Code:**

```python
class ModelRegistry:
    def __init__(self):
        self.models = {
            "model1": {"cost_per_request": 0.5, "max_tokens": 2048},
            "model2": {"cost_per_request": 0.75, "max_tokens": 1024},
        }

class ModelRouter:
    def __init__(self, registry):
        self.registry = registry

    def route_request(self, request):
        available_models = sorted(
            [(model_id, model) for model_id, model in self.registry.models.items()],
            key=lambda x: (x[1]["cost_per_request"], -x[1]["max_tokens"]),
        )
        for model_id, _ in available_models:
            if self.check_model_availability(model_id):
                return model_id

class FallbackManager:
    def handle_overload(self, model_router, request):
        while True:
            selected_model = model_router.route_request(request)
            try:
                response = self.execute_model(selected_model, request)
                return response
            except ModelUnavailableError:
                pass  # Fallback to cheaper models

    def execute_model(self, model_id, request):
        if not self.check_model_availability(model_id):
            raise ModelUnavailableError("Model is unavailable")

class LLMAdapter:
    def __init__(self, model_id):
        self.model = self.load_model(model_id)

    def load_model(self, model_id):
        return ModelProvider().get_model_by_id(model_id)  # Interface to the provider

    def generate_response(self, request):
        response = self.model.generate(request)
        return response
```

**Example input / output:**

```text
registry = ModelRegistry()
router = ModelRouter(registry)
manager = FallbackManager()

request = {"text": "What is the weather like?"}

# Route request to model1
model_id = router.route_request(request)

# Execute model and get response
adapter = LLMAdapter(model_id)
response = adapter.generate_response(request)

print(response)  # Output: {"weather": "sunny"}
```

**Data:**

```json
{
    "models": [
        {"id": "model1", "cost_per_request": 0.5, "max_tokens": 2048},
        {"id": "model2", "cost_per_request": 0.75, "max_tokens": 1024}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Model Available?
5. Rout request to model
6. Generate Response
7. Fallback to cheaper models
8. Rout request to fallback model
9. Generate Response
10. Output Response
11. End

