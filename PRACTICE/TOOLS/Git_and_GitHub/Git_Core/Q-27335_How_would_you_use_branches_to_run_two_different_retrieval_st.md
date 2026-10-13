---
type: "practice"
difficulty: "Intermediate"
---

# How would you use branches to run two different retrieval strategies (keyword vs semantic) in parallel and compare their answers?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git Worktrees](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Worktrees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage running two different retrieval strategies—keyword-based and semantic—in parallel using Git and GitHub, we can leverage feature branches. By branching off a main or stable branch, you can develop each strategy independently without interfering with the other. Once both are ready, you can compare their outputs to determine which one performs better.

#### Triage

The core of this question is about utilizing Git's branching model to isolate and parallelize development efforts for different retrieval strategies. The interviewer may be probing your understanding of branch management, version control best practices, and the ability to manage multiple concurrent developments effectively. Key assumptions include:

You have a stable codebase that you can branch from.
Both keyword-based and semantic retrieval strategies will be developed independently but need to be compared.

#### First Principles Solution
Start with a Stable Base: Begin by forking or cloning the repository where your current project resides.
Create Feature Branches:
Create a keyword-strategy branch from the main/develop branch.
Create a semantic-strategy branch from the same main/develop branch.
Develop Each Strategy Independently:
In the keyword-strategy branch, implement and test your keyword-based retrieval algorithm.
In the semantic-strategy branch, develop and test your semantic-based retrieval algorithm.
Parallel Development: Work on both branches in parallel, ensuring each has a clear feature set and is ready for testing.
Compare Outputs:
Once both strategies are stable, deploy or run tests against the same data to compare their results.
Use Git tools like git diff to compare changes between the two branches if needed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using separate feature branches, you can ensure that each retrieval strategy is developed independently and thoroughly tested. This approach helps in isolating bugs and ensuring a cleaner merge process when integrating the strategies back into your main codebase. Common mistakes include not creating clear naming conventions for branches or failing to document changes properly.

Common follow-ups might involve discussing how to handle merging conflicts, best practices for branch management, and tools that can facilitate comparison (e.g., using GitHub's compare view).

**Comparison:**

```
Keyword Strategy	Semantic Strategy
Implementation
Testing
```

**Comparison:**

```
Integration
```

**Code:**

```
SH
Copy
# Example commands in a Git repository

# Step 1: Create branches from the main/develop branch
git checkout -b keyword-strategy
git checkout -b semantic-strategy

# Step 2: Develop each strategy independently
echo "Implementing keyword-based retrieval..."
echo "Implementing semantic-based retrieval..."

# Step 3: Make changes and commit them to respective branches
git add .
git commit -m "Initial implementation of keyword-based retrieval"
git add .
git commit -m "Initial implementation of semantic-based retrieval"

# Step 4: Compare the two strategies
git diff keyword-strategy semantic-strategy
```

**Example input / output:**

```text
# Example of running tests and comparing outputs

# Test keyword strategy
echo "Running keyword-based retrieval test..."
./run_keyword_tests.sh > keyword_results.txt

# Test semantic strategy
echo "Running semantic-based retrieval test..."
./run_semantic_tests.sh > semantic_results.txt

# Compare the results
diff keyword_results.txt semantic_results.txt
```

**Data:**

```json
{
  "keyword_strategy_data": [
    {"id": 1, "term": "keyword", "score": 0.8},
    {"id": 2, "term": "search", "score": 0.9}
  ],
  "semantic_strategy_data": [
    {"id": 1, "concept": "query", "score": 0.75},
    {"id": 2, "concept": "result", "score": 0.85}
  ]
}
```

**Flow:**

1. Start
2. Fork/Clone Main Repository
3. Yes
4. Create `keyword-strategy`Branch
5. Develop Keyword-BasedStrategy
6. Test Keyword-Based Strategy
7. Commit Changes to`keyword-strategy`
8. Yes
9. Create `semantic-strategy`Branch
10. Develop Semantic-BasedStrategy
11. Test Semantic-Based Strategy
12. Commit Changes to`semantic-strategy`
13. Compare Strategies Using GitTools
14. Merge Strategies Back intoMain/Develop
15. Review and Release

