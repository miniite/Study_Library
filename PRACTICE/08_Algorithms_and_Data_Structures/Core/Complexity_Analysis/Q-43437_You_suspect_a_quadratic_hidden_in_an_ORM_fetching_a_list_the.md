---
type: "practice"
difficulty: "Intermediate"
---

# You suspect a quadratic hidden in an ORM: fetching a list then lazy-loading a relation per item; how do you prove the N+1 is the complexity culprit and fix it while keeping the code readable?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Complexity vs Real Performance](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_vs_Real_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of identifying whether a quadratic hidden in an ORM query is causing "N+1" issues, we need to understand that this issue arises when each item in a list requires an additional database query to fetch related data. The goal is to prove if this behavior is indeed quadratic and then refactor the code while maintaining readability.

#### Triage

The question essentially asks us to:

Identify whether the ORM query is causing N+1 issues.
Prove that it results in a time complexity of O(N^2).
Fix the issue by optimizing the query or refactoring the code, ensuring readability and maintainability.

#### Assumptions
We are using an ORM (Object-Relational Mapping) tool.
The ORM is suspected to be causing N+1 issues when fetching a list of items with related data.
The ORM might not be eager-loading all necessary relations at once.
Step-by-First Principles Solution
Identifying the Issue

To identify if we are dealing with an N+1 query, we can start by observing the number of database queries being executed. If each item in a list requires its own separate query to fetch related data, then it is likely that this is causing performance issues.

We can use profiling tools like SQL Profiler (for databases) or built-in tracing features in ORMs to log and count the number of queries executed. For example:

Count Queries: Use a tool to log all database queries made during the execution.
Profile Execution Time: Measure how long each query takes, especially those that are called multiple times.
Proving Quadratic Complexity

To prove that this is causing quadratic complexity, we can use a simple test case with varying data sizes and observe the performance impact:

Test Case Setup:

Create a list of N items.
Ensure each item has M related entities (e.g., comments or tags).

Execution Time Measurement:

Record the time taken to fetch all items without any lazy loading.
Record the time taken to fetch all items with lazy loading enabled.

Analysis:

If the time taken is approximately proportional to ( N \times M ), then we are likely dealing with an N+1 issue causing quadratic complexity.
Fixing the Issue

Once we have identified and proven the issue, we can fix it by optimizing the query or refactoring the code. Here are a few approaches:

Eager Loading:

Fetch all related data in one go using eager loading.
For example, if we want to fetch users with their comments, we might use something like User.includes(:comments).

Batch Processing:

If eager loading is not possible due to performance constraints, consider batch processing where you first fetch the main entities and then fetch related data in batches or by using pagination.

Subqueries:

Use subqueries to optimize fetching of related data. For example, instead of making N+1 calls, make one call that joins tables efficiently.

Caching:

Implement caching mechanisms like Redis or Memcached to store frequently accessed related entities and reduce database hits.
Code Example

Here's a simplified example in Ruby on Rails:

RUBY
# Original code with N+1 issue
users = User.all
users.each do |user|
  comments = user.comments
end

# Optimized code with eager loading
users_with_comments = User.includes(:comments).all
Full Code Example
RUBY
class UserController < ApplicationController
  def index
    # Original implementation with N+1 issue
    # users = User.all
    # render json: { users: users.map { |u| u.to_json(include: :comments) } }

    # Optimized implementation with eager loading
    users_with_comments = User.includes(:comments).all
    render json: { users: users_with_comments.map { |u| u.to_json(include: :comments) } }
  end
end

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to address the N+1 issue, we first need to identify if it is causing quadratic complexity by profiling database queries. We then can fix this issue by optimizing the query using techniques like eager loading, batch processing, or caching.

#### Common mistakes
include:

Not identifying the N+1 issue correctly.
Overcomplicating solutions leading to less maintainable code.
Ignoring performance issues in favor of simplicity.

#### Follow-ups
might involve discussing specific tools and libraries used for profiling and optimization, as well as trade-offs between different approaches.

Self-Check

Correctness + completeness confirmed.

