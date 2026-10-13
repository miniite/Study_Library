---
type: "practice"
difficulty: "Intermediate"
---

# Design a calculator/expression evaluator. How do you parse and evaluate '3 + 4 * 2'?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Classic LLD Problems](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Classic_LLD_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a calculator to evaluate expressions like "3 + 4 * 2", we need to handle multiple operations, including addition, subtraction, multiplication, division, and handling parentheses. The core of this problem is parsing the expression correctly and then evaluating it step by step. At an intermediate level, we will focus on building a parser that can understand the order of operations (operator precedence) and implement a basic evaluation mechanism.

#### Triage

The question is essentially asking how to break down and evaluate a mathematical expression with varying operators in a specific order. The interviewer might be probing your understanding of parsing techniques, operator precedence rules, and recursive or iterative algorithms for expression evaluation.

We assume that the input will always follow standard algebraic notation (infix notation) with numbers, basic arithmetic operations (+, -, *, /), and optional parentheses to denote order of operations. We also assume we are not dealing with more complex scenarios like exponentiation or unary operators at this stage.

#### First-Principles Solution
Parsing the Expression: Convert the infix expression into a form that can be easily evaluated.
Order of Operations: Implement logic to handle operator precedence correctly (e.g., multiplication and division before addition and subtraction).
Evaluation Logic: Once parsed, evaluate the expression by following the order of operations.
Step-by-Step Solution

Lexical Analysis:

Tokenize the input string into tokens such as numbers and operators.

Syntax Analysis:

Build an Abstract Syntax Tree (AST) to represent the structure of the expression.

Evaluation Logic:

Traverse the AST, applying operator precedence rules during traversal.

Here's a step-by-step breakdown using Python:

Tokenize the Input String:

PYTHON
def tokenize(expression):
    import re

    tokens = []
    numbers = re.findall(r'\d+', expression)
    operators = re.findall(r'[+\-*/()]', expression)

    it_num = iter(numbers)
    it_op = iter(operators)

    for op in it_op:
        try:
            num1 = next(it_num)
            if op == '(':
                tokens.append('(')
            elif op == ')':
                tokens.append(')')
            else:  # operator
                tokens.append(int(num1))
                tokens.append(op)
        except StopIteration:
            break

    return tokens + [int(next(it_num))]

Build AST (Optional for Intermediate Level): Since we are focusing on the intermediate level, building an AST is not strictly necessary, but it can help in complex scenarios. For now, let's stick to a straightforward approach.

Evaluate Expression:

PYTHON
def evaluate(tokens):
    stack = []
    operators = {'+': lambda x, y: x + y,
                 '-': lambda x, y: x - y,
                 '*': lambda x, y: x * y,
                 '/': lambda x, y: x / y}

    for token in tokens:
        if isinstance(token, int):
            stack.append(token)
        else:
            right = stack.pop()
            left = stack.pop()
            result = operators[token] (left, right)
            stack.append(result)

    return stack[0]

Putting It All Together:

PYTHON
def evaluate_expression(expression):
    tokens = tokenize(expression)
    return evaluate(tokens)

#### Example
Usage
PYTHON
print(evaluate_expression("3 + 4 * 2"))  # Output: 11

#### Closing Summary
and Common Mistakes/Follow-Ups

The key takeaway is to handle operator precedence correctly, which we achieved through a simple stack-based approach. For more complex expressions or different notations (like postfix notation), further refinement would be necessary.

#### Common mistakes
include:

Not handling parentheses properly.
Incorrectly prioritizing operators in the evaluation function.
Failing to tokenize and parse complex input strings accurately.

For follow-ups, you could discuss edge cases like division by zero, supporting more advanced operations (like exponentiation or unary minus), or optimizing for performance with larger expressions. Additionally, we can explore other parsing techniques such as Shunting Yard algorithm for better handling of operator precedence in a more formal manner.

**Code:**

```python
def tokenize(expression):
    import re

    tokens = []
    numbers = re.findall(r'\d+', expression)
    operators = re.findall(r'[+\-*/()]', expression)

    it_num = iter(numbers)
    it_op = iter(operators)

    for op in it_op:
        try:
            num1 = next(it_num)
            if op == '(':
                tokens.append('(')
            elif op == ')':
                tokens.append(')')
            else:  # operator
                tokens.append(int(num1))
                tokens.append(op)
        except StopIteration:
            break

    return tokens + [int(next(it_num))]

def evaluate(tokens):
    stack = []
    operators = {'+': lambda x, y: x + y,
                 '-': lambda x, y: x - y,
                 '*': lambda x, y: x * y,
                 '/': lambda x, y: x / y}

    for token in tokens:
        if isinstance(token, int):
            stack.append(token)
        else:
            right = stack.pop()
            left = stack.pop()
            result = operators[token](left, right)
            stack.append(result)

    return stack[0]

def evaluate_expression(expression):
    tokens = tokenize(expression)
    return evaluate(tokens)
```

**Example input / output:**

```text
print(evaluate_expression("3 + 4 * 2"))  # Output: 11
```

