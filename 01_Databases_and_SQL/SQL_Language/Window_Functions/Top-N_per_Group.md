---
id: "01_Databases_and_SQL/SQL_Language/Window_Functions/Top-N_per_Group"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Window_Functions"
tools: []
---

# Top-N per Group

**L4 Advanced** · [↑ Window Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** To get the top N rows (or the Nth value) within each group, rank the rows with a window function partitioned by the group and filter on the rank in an outer query.

### 1. The big picture
"Highest-paid employee per department", "top 3 products per category", "second-highest salary per department" all share one shape: ranking *within* groups, then keeping the first N. `TOP (N)` or `LIMIT N` only cut the whole result, not each group. The window function solution partitions by the group, orders by the measure, ranks, then filters.

The same family also includes "compare each row to its group": value minus group average, share of group total, max in the group — all window aggregates with PARTITION BY and no ORDER BY. They keep every row, which is why they belong to this lesson's pattern set.

### 2. Core concepts and vocabulary
- **Top-N per group** — N best rows in each partition.
- **Rank basis** — ROW_NUMBER (exactly N rows), RANK (ties included, gaps), DENSE_RANK (N distinct values).
- **Group aggregate on every row** — `AVG(x) OVER (PARTITION BY g)`.
- **Qualify** — filter on a window result (`QUALIFY` in Snowflake/BigQuery/Teradata; CTE elsewhere).
- **APPLY / LATERAL** — runs a subquery per outer row, an alternative for top-N.
- **Tiebreaker** — second ORDER BY column to make results stable.

### 3. How it works, step by step
1. Decide what a tie at the cut-off should do: keep exactly N rows, or keep all tied rows?
2. Pick ROW_NUMBER (exactly N), RANK (N positions incl. ties), or DENSE_RANK (N distinct values).
3. Compute it in a CTE with `PARTITION BY group ORDER BY measure DESC, tiebreaker`.
4. Filter `rank <= N` outside.
5. Add a final ORDER BY for presentation.

### 4. Worked example
```sql
-- highest-paid employee per department (ties: keep all)
WITH r AS (
  SELECT *, RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rnk FROM dbo.Employees)
SELECT * FROM r WHERE rnk = 1;

-- top 3 per department, exactly 3 rows each
WITH r AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC, empid) AS rn FROM dbo.Employees)
SELECT * FROM r WHERE rn <= 3;

-- second-highest salary per department (value)
WITH r AS (
  SELECT dept, salary, DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS d
  FROM dbo.Employees)
SELECT DISTINCT dept, salary FROM r WHERE d = 2;

-- every employee beside group context, no collapse
SELECT empid, dept, salary,
       AVG(salary) OVER ()                              AS company_avg,
       MIN(salary) OVER (PARTITION BY dept)             AS dept_min,
       MAX(salary) OVER (PARTITION BY dept)             AS dept_max,
       salary - AVG(salary) OVER (PARTITION BY dept)    AS diff_from_dept_avg,
       100.0 * salary / SUM(salary) OVER (PARTITION BY dept) AS pct_of_dept,
       CASE WHEN salary > AVG(salary) OVER () THEN 'above' ELSE 'below' END AS band,
       CASE WHEN RANK() OVER (PARTITION BY dept ORDER BY salary DESC) = 1
            THEN 'Top earner' END AS label
FROM dbo.Employees;
```
Alternative for few groups with an index on `(dept, salary DESC)`:
```sql
SELECT d.dept, t.*
FROM dbo.Departments d
CROSS APPLY (SELECT TOP (3) * FROM dbo.Employees e
             WHERE e.dept = d.dept ORDER BY e.salary DESC, e.empid) t;
```
Median per department: `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER (PARTITION BY dept)`.

### 5. Under the hood
- The window plan sorts the whole table by `(dept, salary DESC)`, numbers it, then a Filter drops all but N rows per group. The sort touches **every** row even though few survive.
- With a supporting index the sort disappears but the numbering still scans the entire index.
- `CROSS APPLY TOP (N)` becomes a nested-loops plan with one index seek per group: cost ∝ groups × N. It wins when groups are few and big; the window version wins when groups are many and small.
- `QUALIFY`-capable engines run the filter in the same step but conceptually the same plan.
- Window aggregates without ORDER BY compute the group value once per partition and replicate it, via a Window Spool in SQL Server (or hash aggregate + join in others).

### 6. Performance and design implications
- Index `(group, measure DESC) INCLUDE (needed columns)`; the optimizer can then stream the ranking.
- Filter early (`WHERE` on date, status) so fewer rows reach the sort.
- For huge fact tables, pre-aggregate then rank (top products by revenue per category from 1M rows, not 1B).
- In distributed engines, a skewed group (one department with most rows) creates a hot task; consider two-phase top-N (local top-N then global).
- Decide ties before coding; "top 3" without a rule is ambiguous and a business question.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ROW_NUMBER() ... WHERE rn <= N` | Exactly N per group | Fixed-size lists |
| `RANK()` / `DENSE_RANK()` | Include ties / distinct values | Fair top-N, Nth highest |
| `TOP (N) WITH TIES` | Overall top N with ties | Not per group |
| `CROSS APPLY` / `LATERAL` | Per-group subquery | Few groups, good index |
| `QUALIFY` | Filter on window in same SELECT | Snowflake/BigQuery |
| `AVG/MAX/MIN() OVER (PARTITION BY g)` | Group stat on each row | Compare row to group |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Window ranking: many groups, general purpose | Few groups, huge rows each → APPLY + index |
| ROW_NUMBER for strict N | Ties matter → RANK/DENSE_RANK |
| Window aggregate beside row | Only the group totals are needed → GROUP BY |

### 9. Common mistakes
- `SELECT TOP 3 ... GROUP BY dept` expecting per-group → returns 3 rows total → use ranking.
- Using ROW_NUMBER and silently dropping tied rows → unfair results → choose RANK or document the tiebreak.
- RANK for "second highest" when top is tied → no rank 2 → DENSE_RANK.
- Filtering the rank in the same SELECT → error → CTE/subquery.
- Joining aggregated subquery back to detail (self-join) when a window would do → double scan → window aggregate.
- Integer division in percent-of-total → 0 → multiply by 100.0.

### 10. 🎯 Interview kit
**30-second answer:** Partition by the group, order by the measure descending, assign RANK/ROW_NUMBER/DENSE_RANK, and filter `<= N` in an outer query. Choose the function by how ties should behave.

**Structure a longer answer:** clarify ties and N → ranking pattern → alternative (APPLY) → index and sort cost → scale (pre-aggregate, skew).

**Likely follow-ups:**
- *Why not GROUP BY?* → it returns aggregates, not the rows themselves.
- *ROW_NUMBER vs RANK here?* → exactly N vs include ties.
- *How do you make it fast?* → index on group + measure, filter early, APPLY for few groups.
- *Median per group?* → PERCENTILE_CONT or ranking with counts.

**Signals interviewers look for:** tie-handling discussion, knowing the CTE requirement, alternatives with trade-offs.

### 11. Check yourself
- [ ] Write top-2 per category with ties included and with ties excluded.
- [ ] Write second-highest per department using DENSE_RANK and by OFFSET/FETCH.
- [ ] Show the same result via CROSS APPLY and compare plans.
- [ ] Produce each row's percentage of its department total.
<!-- explainer:end -->

## Practice questions

- [Basic] [Write a query that adds a column showing the average salary across the whole table next to every employee row.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44158_Write_a_query_that_adds_a_column_showing_the_average_salary_.md>)
- [Basic] [How would you show, for each employee, how their salary compares to the maximum salary in their team?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44176_How_would_you_show_for_each_employee_how_their_salary_compar.md>)
- [Intermediate] [How do you find the second highest salary in each department using a window function?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44332_How_do_you_find_the_second_highest_salary_in_each_department.md>)
- [Intermediate] [How would you compute the difference between each employee's salary and the department average on the same row?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44403_How_would_you_compute_the_difference_between_each_employee_s.md>)
- [Intermediate] [How would you find, for each employee, the salary of the highest-paid person in their department using MAX() OVER?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44422_How_would_you_find_for_each_employee_the_salary_of_the_highe.md>)
- [Intermediate] [Write a query that flags each row as 'above average' or 'below average' compared to the overall average salary.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44189_Write_a_query_that_flags_each_row_as_above_average_or_below_.md>)
- [Intermediate] [Write a query to compute the second-highest salary per department (not overall).](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44195_Write_a_query_to_compute_the_second-highest_salary_per_depar.md>)
- [Intermediate] [Write a query combining a CASE expression with a window function to label the top earner in each department.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Window_Functions/Q-44219_Write_a_query_combining_a_CASE_expression_with_a_window_func.md>)

## Related concepts

- [Subquery Practice Problems](<../Joins_and_Subqueries/Subquery_Practice_Problems.md>) *(Joins and Subqueries)*
- [Aggregate Precision and Types](<../Aggregation/Aggregate_Precision_and_Types.md>) *(Aggregation)*
- [Basic Query Practice](<../../../TOOLS/MySQL/MySQL_Platform/Basic_Query_Practice.md>) *(MySQL Platform)*

