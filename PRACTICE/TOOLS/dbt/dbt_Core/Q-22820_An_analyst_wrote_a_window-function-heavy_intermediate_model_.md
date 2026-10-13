---
type: "practice"
difficulty: "Intermediate"
---

# An analyst wrote a window-function-heavy intermediate model that's now the slowest node in the DAG. How do you profile and optimize it while keeping logic readable?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ dbt Models and Runs](<../../../../TOOLS/dbt/dbt_Core/dbt_Models_and_Runs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this challenge, we'll first need to understand what the analyst has done with their window functions-heavy model. The key is to balance performance optimization with maintaining readability of the code. We will use dbt's profiler and our domain knowledge to identify bottlenecks and refactor the model.

Triaging the Problem

The question is asking how to efficiently profile a specific node in a dbt DAG that uses extensive window functions, ensuring that we don’t compromise on code clarity while significantly improving performance. The core of the solution involves understanding the current logic, identifying slow parts using profiling tools, and then making targeted optimizations.

#### Assumptions
Current Model: The analyst’s model is complex, with multiple window functions.
Performance Issue: This node has become the bottleneck in the DAG, significantly slowing down the overall build process.
Readability vs Performance: We need to maintain a good balance between performance and code readability.

#### First-Principles Solution

Profile the Model:

Use dbt run-operation with a custom operation that collects profiling data. This allows us to understand how long each subquery takes.
SQL
-- Custom Profiling Operation
select * from dbt_profiler_your_model()

Identify Bottlenecks:

Analyze the collected profiles to find which window functions are taking the most time.
Identify any redundant or overly complex expressions within the window functions.

Optimize Window Functions:

Simplify complex window functions by breaking them down into smaller, reusable subqueries.
Use more efficient window function parameters and arguments if possible.

Refactor Code for Readability:

Maintain a clean, modular structure that is easy to understand but also performant.
Document the rationale behind any changes made to ensure maintainability.

Test and Iterate:

After refactoring, run thorough testing to ensure that the model still produces correct results.
Continuously profile after each iteration until performance meets expectations.

#### Closing Summary
+ Common Mistakes

By following these steps, we can effectively optimize the window-function-heavy node while keeping the logic readable. The key is to use profiling tools judiciously and make incremental changes that balance performance with maintainability. Common mistakes include over-optimizing early in the process without proper profiling or failing to document the reasoning behind each change.

#### Example
of Profiling and Optimization

Let's assume we have a complex window function that calculates rolling averages, which is taking too long:

Original Code:
SQL
-- Original Model
select
  id,
  date,
  avg(value) over (partition by category order by date rows between 3 preceding and current row) as rolling_avg
from your_table
Optimized Code:
Break Down into Subqueries:
SQL
-- Subquery for Rolling Average Calculation
with rolling_averages as (
  select
    id,
    date,
    value,
    lag(value, 3) over (partition by category order by date) as prev_value_1,
    lag(value, 2) over (partition by category order by date) as prev_value_2,
    lag(value, 1) over (partition by category order by date) as prev_value_3
  from your_table
)
-- Final Calculation
select
  id,
  date,
  (prev_value_1 + prev_value_2 + prev_value_3 + value) / 4 as rolling_avg
from rolling_averages

By breaking down the window function, we reduce the complexity and make it easier to understand. This approach also helps in identifying any redundant calculations.

Profiling Output Example:
PLAINTEXT
-- Sample Profiling Data
Node: your_model
Time Taken: 120 seconds

- Subquery 'rolling_averages': 80 seconds (67% of total time)
- Final Calculation: 40 seconds (33% of total time)
Self-check:

This solution is correct and complete.

**Comparison:**

```
Start Profiling
Run Custom Operation
Analyze Profiles
Identify Bottlenecks
Optimize Window Functions
Test and Iterate
Profile Again
Repeat if Necessary
```

**Code:**

```

-- Custom Profiling Operation select * from dbt_profiler_your_model()

CODE
Copy
```sql
with rolling_averages as (
  select
    id,
    date,
    value,
    lag(value, 3) over (partition by category order by date) as prev_value_1,
    lag(value, 2) over (partition by category order by date) as prev_value_2,
    lag(value, 1) over (partition by category order by date) as prev_value_3
  from your_table
)
select
  id,
  date,
  (prev_value_1 + prev_value_2 + prev_value_3 + value) / 4 as rolling_avg
from rolling_averages
```

**Example input / output:**

```text
-- Profiling Output Example
Node: your_model
Time Taken: 120 seconds

- Subquery 'rolling_averages': 80 seconds (67% of total time)
- Final Calculation: 40 seconds (33% of total time)

Optimized Code:
Node: your_model_optimized
Time Taken: 50 seconds
```

**Data:**

```sql
-- Sample Data for Profiling and Optimization
id | date       | category | value
---|------------|----------|------
1  | 2023-01-01 | A        | 10
1  | 2023-01-02 | A        | 20
1  | 2023-01-03 | A        | 30
1  | 2023-01-04 | A        | 40
```

**Flow:**

1. Start Profiling
2. Run Custom Operation
3. Analyze Profiles
4. Identify Bottlenecks
5. Optimize Window Functions
6. Test and Iterate
7. Profile Again
8. Repeat if Necessary

