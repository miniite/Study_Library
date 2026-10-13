---
type: "practice"
difficulty: "Intermediate"
---

# Which pattern would you use to represent the parsed expression tree and why is it the Composite/Interpreter?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Over-engineering vs Simplicity](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a low-level representation of an expression tree, we often choose between several patterns, but in this case, the Composite and Interpreter patterns are both strong contenders. The core of the question is to understand which pattern best suits the task and why one might be preferred over the other.

#### Triaging the Question

The interviewer is likely probing your understanding of both design patterns, their applicability, and how they can be used in practice to model complex data structures such as expression trees. Key assumptions here include:

The problem context involves building a system that needs to handle different types of expressions (e.g., arithmetic or logical).
You are required to represent these expressions hierarchically, allowing for both atomic elements and composite structures.

#### First-Principles Solution
Composite Pattern

The Composite pattern allows us to treat individual objects and compositions of those objects uniformly. It is ideal when we need to handle a collection of objects as if they were single objects. In the context of expression trees, this means being able to treat both simple values (like numbers or variables) and complex expressions (like arithmetic operations) in a consistent manner.

Interpreter Pattern

The Interpreter pattern involves constructing an interpreter for a given language, typically represented by an abstract syntax tree (AST). This pattern is used when you want to define a small domain-specific language and interpret that language within your application. For expression trees, this means implementing rules or operations defined by the language of expressions.

Why Composite?
Uniform Interface: The Composite pattern provides a uniform interface for handling both simple and complex expressions. This makes it easier to implement algorithms that can operate on any type of expression without needing to know its specific structure.
Hierarchical Structure: It naturally supports hierarchical structures, where an expression might be composed of multiple sub-expressions. For example, in the expression (3 + 4) * (5 - 1), both + and - are composite operations themselves.
Why Interpreter?
Domain-Specific Language: The Interpreter pattern is perfect for situations where you want to evaluate expressions that define a specific language or set of rules. For example, in a mathematical expression evaluator, you might have different types of operators (addition, multiplication) and values.
Rule-Based Interpretation: Each node in the tree can represent not just data but also actions or rules. This is particularly useful when the evaluation logic itself needs to be represented within the structure.

#### Closing Summary

In summary, both Composite and Interpreter patterns have their merits. The choice between them depends on whether your focus is more on handling a uniform interface for hierarchical structures (Composite) or implementing specific domain rules through an interpreted language (Interpreter). For most typical expression tree scenarios, the Composite pattern would be the preferred choice due to its simplicity and broader applicability.

#### Common Mistakes/Follow-Ups
Misunderstanding Hierarchical Structures: Thinking that only one pattern can handle hierarchical data structures.
Ignoring Domain-Specific Logic: Failing to recognize when specific rules or logic need to be embedded in the tree structure, which is where Interpreter shines.

The interviewer might follow up with questions like:

How would you implement an expression evaluator using both patterns?
What are potential drawbacks of each approach?

**Comparison:**

```
Pattern	Type	Key Features
Composite	Abstract Syntax Tree (AST)	Hierarchical structure, uniform interface for objects and their sub-elements.
Interpreter	Domain-Specific Language	Evaluates expressions using specific rules defined within the tree structure.
```

**Code:**

```python
class Expression:
    def evaluate(self):
        pass

class Number(Expression):
    def __init__(self, value):
        self.value = value

    def evaluate(self):
        return self.value

class BinaryExpression(Expression):
    def __init__(self, left, right, operator):
        self.left = left
        self.right = right
        self.operator = operator

    def evaluate(self):
        if self.operator == '+':
            return self.left.evaluate() + self.right.evaluate()
        elif self.operator == '-':
            return self.left.evaluate() - self.right.evaluate()
        # Add more operators as needed

# Example usage with Composite pattern
expr1 = Number(3)
expr2 = Number(4)
expr3 = BinaryExpression(expr1, expr2, '+')

result = expr3.evaluate()  # result will be 7
```

**Example input / output:**

```text
Input:
- Expression tree with `3 + 4`

Output:
- Result of evaluation is `7`
```

**Data:**

```json
{
    "expression": {
        "type": "BinaryExpression",
        "left": {"value": 3},
        "right": {"value": 4},
        "operator": "+"
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Expression Node
8. Is Leaf?
9. Number
10. BinaryExpression
11. Left Sub-expression?
12. Expression Node
13. BinaryExpression
14. Right Sub-expression?
15. Expression Node
16. BinaryExpression

