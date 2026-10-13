---
type: "practice"
difficulty: "Basic"
---

# What context does Cursor use from your notebook or script to give better ML code suggestions than a plain chatbot?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Onboarding to Codebases with AI](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Onboarding_to_Codebases_with_AI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with AI-assisted coding tools like "Cursor," the primary goal is to provide highly relevant and contextually accurate code suggestions based on the current state of your notebook or script. The key idea here is that these tools leverage a deep understanding of the surrounding context, which goes beyond simple keyword matching used by traditional chatbots.

#### Triaging the Question

The question is essentially asking about the underlying mechanisms Cursor uses to provide better ML code suggestions compared to a plain chatbot. This involves understanding how context from the notebook or script influences the tool's decision-making process and what types of information are most valuable for generating accurate suggestions.

#### First-Principles Solution
Contextual Understanding: The primary source of context is the current state of your code, including variables, functions, data structures, and the overall workflow.
Code Analysis: Cursor analyzes the syntax, semantics, and structure of the code to understand what you are currently working on.
Knowledge Base Integration: It integrates this contextual understanding with a vast knowledge base that includes programming best practices, common patterns in machine learning (ML) and data science, and even real-world examples from similar projects.
Detailed Explanation
1. Contextual Understanding
Variable Scope: Cursor can infer the types of variables being used and their current scope. For example, if you are working on a function where X is a DataFrame, it knows that any code suggestions should be relevant to data manipulation.
Function Definitions: It understands the structure and parameters of functions or methods defined in your notebook. This helps in suggesting more appropriate function calls or method arguments.
2. Code Analysis
Syntax and Semantics: By parsing the code, Cursor can understand the intended operations and propose suggestions that align with these intentions.
Type Inference: It uses type inference to suggest types for variables or parameters, ensuring the suggested code is syntactically correct and semantically meaningful.
3. Knowledge Base Integration
Best Practices and Patterns: The knowledge base contains a wealth of information on best practices in ML and data science, including common algorithms, techniques, and patterns.
Real-World Examples: It leverages examples from real-world projects to provide more concrete suggestions that are relevant to the current context.

#### Example Scenario

Suppose you have a Python notebook with some initial code:

PYTHON
import pandas as pd
from sklearn.model_selection import train_test_split

# Load data
data = pd.read_csv('data.csv')

# Split data into training and testing sets
X, y = train_test_split(data)

If you then start typing model, Cursor might suggest RandomForestClassifier or LogisticRegression, given the context of splitting the dataset. This is significantly more relevant than a chatbot's random suggestions.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, Cursor uses the current state of your code and integrates it with a comprehensive knowledge base to provide highly contextual and accurate ML coding suggestions. The key mistakes often arise from underestimating how deeply context is analyzed (just syntax) or overestimating the generalizability of chatbot responses.

#### Common Mistakes

Focusing solely on keyword matching without understanding the actual code structure.
Ignoring the importance of real-world examples and best practices in ML.

#### Follow-ups
could include discussions about how Cursor handles edge cases, how it balances between suggesting too many options versus not enough, or exploring alternative approaches to integrating contextual information.

