---
type: "practice"
difficulty: "Intermediate"
---

# Design a validation framework where rules are composable. Which patterns keep it extensible?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Extensible Rule Engines](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Extensible_Rule_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a validation framework that allows rules to be composably, we aim for a system that is modular, flexible, and easy to extend or modify without affecting other parts of the application. The key patterns that make such a system extensible include Strategy, Decorator, Chain of Responsibility, and Composite design patterns. This answer will break down each pattern's role in making our validation framework robust and flexible.

The core idea is to separate concerns, allow rules to be combined dynamically, and ensure that adding new validation logic does not require changes throughout the existing codebase.

**Comparison:**

```
Pattern	Description	Example
Strategy	Define a family of algorithms, encapsulate each one, and make them interchangeable.	Custom validators for different validation types.
Decorator	Attach new responsibilities to an object dynamically; decorators provide a flexible alternative to subclassing for extending functionality.	Adding logging or debugging hooks around validation rules.
Chain of Responsibility	Pass the request along a chain of potential handlers until one of them handles it, allowing requests to pass through a series of validators without knowing which will handle them next.	Sequential execution of multiple validation steps in different services.
Composite	Treat individual objects and compositions of objects uniformly; allows for hierarchical structures.	Managing nested rules that can be added or removed dynamically.
```

**Code:**

```python
from abc import ABC, abstractmethod

class Rule(ABC):
    @abstractmethod
    def validate(self, data) -> bool:
        pass

class CompositeRule(Rule):
    def __init__(self, rules=None):
        if rules is None:
            self.rules = []
        else:
            self.rules = rules

    def add_rule(self, rule: Rule):
        self.rules.append(rule)

    def validate(self, data) -> bool:
        for rule in self.rules:
            if not rule.validate(data):
                return False
        return True

class DecoratorRule(Rule):
    def __init__(self, wrapped_rule: Rule):
        self.wrapped_rule = wrapped_rule

    def validate(self, data) -> bool:
        return self.wrapped_rule.validate(data)
```

**Example input / output:**

```

To illustrate, let's create a composite rule that combines an email validation and length validation:

PYTHON
Copy
email_rule = EmailRule("@example.com")
length_rule = LengthRule(5, 20)
composite_rule = CompositeRule([email_rule, length_rule])
valid_data = "test@example.com"
invalid_data = "te@.com"

# Output: True for valid data, False for invalid data
print(composite_rule.validate(valid_data))  # True
print(composite_rule.validate(invalid_data))  # False
```

**Data:**

```json
{
  "rules": [
    {
      "type": "EmailRule",
      "value": "@example.com"
    },
    {
      "type": "LengthRule",
      "min_length": 5,
      "max_length": 20
    }
  ]
}
```

**Flow:**

1. CompositeRule
2. EmailRule
3. LengthRule
4. Result
5. Result

