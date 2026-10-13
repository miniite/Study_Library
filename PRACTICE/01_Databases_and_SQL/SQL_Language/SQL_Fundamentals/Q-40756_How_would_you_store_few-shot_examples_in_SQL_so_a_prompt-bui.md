---
type: "practice"
difficulty: "Intermediate"
---

# How would you store few-shot examples in SQL so a prompt-builder can fetch the k most relevant examples for a given input?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Experiment Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Experiment_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store few-shot examples in SQL and ensure they are fetched efficiently, we need to design a table schema that supports fast retrieval of the k most relevant examples based on a given input. This involves understanding how the prompt-builder works, identifying relevant columns, and choosing appropriate indexing strategies.

The core idea is to create an examples table with relevant fields such as id, input, output, relevance_score, and any other metadata that might be useful for ranking. The key challenge is to enable quick fetching of the top k examples based on a given input, which requires an efficient indexing strategy.

#### Triage

The question asks us to design a SQL schema and query plan for storing and retrieving few-shot examples efficiently. The interviewer is probing our understanding of database optimization techniques such as indexes, query performance, and data structures that can support fast retrieval.

#### Assumptions
We have a clear notion of what "relevance" means in the context of these examples.
The prompt-builder needs to fetch k most relevant examples for any given input.
There might be multiple ways to score relevance (e.g., cosine similarity, edit distance).

#### First Principles Solution

Table Schema: We will create an examples table with the following columns:

id: A unique identifier for each example.
input: The input string or data point used in the prompt.
output: The corresponding output or response generated from the input.
relevance_score: A numerical value indicating how relevant an example is to a given input.

Indexing: To enable fast retrieval, we will create indexes on the input and relevance_score columns. Specifically:

An index on (input) to quickly filter examples by the input.
A composite index on (input, relevance_score DESC) or a full-text search index if the inputs are text-heavy.

Query Strategy: For fetching the k most relevant examples for a given input, we can use a combination of indexing and ranking techniques:

SQL
SELECT id, input, output, relevance_score
FROM examples
WHERE input = 'given_input'
ORDER BY relevance_score DESC
LIMIT k;

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures efficient retrieval by leveraging indexes to quickly filter the relevant rows and then ordering them based on the relevance_score. However, there are several common pitfalls:

Index Selection: Choosing the right columns for indexing is crucial; selecting too many or irrelevant columns can degrade performance.
Relevance Calculation: The method of calculating relevance should be well-defined to ensure meaningful results. For text inputs, techniques like cosine similarity or TF-IDF might be useful.
Scaling: As the number of examples grows, more sophisticated ranking and caching strategies might be necessary.
Common Follow-Ups
What if there are multiple relevant scores for a given input?
How would you handle real-time updates to relevance scores?
Can we optimize this further with materialized views or other database features?

