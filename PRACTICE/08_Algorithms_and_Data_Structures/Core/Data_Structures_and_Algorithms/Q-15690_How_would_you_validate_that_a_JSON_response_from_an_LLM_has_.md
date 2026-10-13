---
type: "practice"
difficulty: "Basic"
---

# How would you validate that a JSON response from an LLM has balanced braces before parsing, and which data structure checks bracket balance?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Recursion vs Explicit Stack](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_vs_Explicit_Stack.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a JSON response from a language model (LLM) has balanced braces, we need to validate the syntax before attempting to parse it. This involves checking if every opening brace { and ( has a corresponding closing brace } and ). A stack-based data structure is well-suited for this task because it follows the Last-In-First-Out (LIFO) principle, making it easy to keep track of the opening braces.

In essence, we are probing your understanding of fundamental data structures and their application in solving real-world problems. Let's break down the solution step by step:

Understand the Problem: We need to validate if a string containing curly braces {} is balanced.
Identify Assumptions: Assume that the input string only contains these characters: { and }.

#### First-Principles Solution
Initialize an empty stack.
Iterate through each character in the string.
If the character is an opening brace ({), push it onto the stack.
If the character is a closing brace (}), check if the stack is not empty. If it is not, pop the top element from the stack (which should be a corresponding opening brace). If the stack is empty, return false because there's no corresponding opening brace for the current closing brace.
Final Check: After iterating through all characters, if the stack is empty, the string is balanced; otherwise, it is not.

This approach ensures that we can handle any nested or interleaved braces correctly. Let's go through a dry run with an example:

JSON
Input: {"key": {"nested": [1, 2, 3]}}
Start with an empty stack.
Iterate and push { onto the stack for "key": {.
Push another { for {"nested": {.
Pop a { from the stack for closing brace }}.
Continue until the end of the string. The stack should be empty at the end, indicating balanced braces.

