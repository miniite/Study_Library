---
id: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Ragged_Hierarchies"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling"
tools: []
---

# Ragged Hierarchies

**L4 Advanced** · [↑ Dimensional Modeling](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A ragged hierarchy is a tree whose branches have different depths (or skip levels); fixed-column dimension tables cannot hold it cleanly, so you fill the gaps, or use a parent-child structure with a bridge table.

### 1. The big picture
Many dimensions are hierarchies: product → subcategory → category, country → state → city. When every path has the same number of levels, you flatten it into one wide dimension row with a column per level — the standard star-schema approach. Real organisations are messier. A company org chart has some managers four levels above a worker and others six. A geography has countries with no states, or a city that rolls straight to a country. Such trees are **ragged** (variable depth) or **unbalanced**.

The challenge: SQL group-by works on fixed columns, but the data has an arbitrary number of levels. There are three families of answer: *pad the gaps*, *flatten with repeated values*, or *represent the tree explicitly with parent-child links and a bridge/closure table*.

### 2. Core concepts and vocabulary
- **Hierarchy** — a tree of parent-child relationships within one dimension; each member has at most one parent.
- **Balanced hierarchy** — every leaf is at the same depth (fixed levels); fits flat columns.
- **Unbalanced / ragged hierarchy** — leaves at different depths, or levels skipped.
- **Parent-child (adjacency list)** — each row stores its parent's key; naturally handles any depth.
- **Level columns** — `level_1 … level_n` columns holding ancestor names, with a defined maximum depth.
- **Path / materialized path** — a string such as `/1/4/9` holding the ancestors from root to the node.
- **Bridge (closure) table** — one row per ancestor-descendant pair, with distance, so any roll-up is a join and group-by.
- **Recursive CTE** — SQL construct that walks an adjacency list to any depth at query time.
- **Rollup** — aggregating leaf measures up to higher levels (`ROLLUP` in `GROUP BY` for fixed levels).

### 3. How it works, step by step
**Technique A — pad the gaps (flat levels)**
1. Decide the maximum depth, say 5 levels.
2. For each member, fill the levels above it with its ancestors, and for levels *below* its own level repeat the member's own value.
3. Queries group by any level column and get a complete rollup; short branches simply repeat.

**Technique B — parent-child with recursive CTE**
1. Keep `member_key`, `parent_key`, `name`.
2. Walk down from a root using a recursive CTE to find all descendants.
3. Join descendants to facts and sum. Fine for small trees; slow and inconvenient for BI tools.

**Technique C — bridge/closure table**
1. For every node, insert rows (ancestor, descendant, depth) for itself and each ancestor.
2. Fact → bridge on descendant key → group by ancestor.
3. Any hierarchy depth, any level is a simple join; you can also store a `top_flag` and `is_leaf_flag` to avoid double counting.

| Technique | Depth flexibility | Query simplicity | Maintenance |
|---|---|---|---|
| Padded level columns | fixed max | very simple, BI-friendly | rebuild on change |
| Adjacency list + recursive CTE | unlimited | needs recursion | easy to edit |
| Materialized path | unlimited | `LIKE 'path%'` | moderate |
| Closure/bridge table | unlimited | simple joins | extra table, rebuild on change |

### 4. Worked example
An employee org with ragged depth, handled with a bridge table.

```sql
CREATE TABLE dim_employee (employee_key int PRIMARY KEY, name varchar(60), manager_key int NULL);

-- closure table: every (ancestor, descendant) pair, including self at depth 0
WITH h AS (
    SELECT employee_key AS ancestor_key, employee_key AS descendant_key, 0 AS depth
    FROM dim_employee
    UNION ALL
    SELECT h.ancestor_key, e.employee_key, h.depth + 1
    FROM h JOIN dim_employee e ON e.manager_key = h.descendant_key
)
SELECT ancestor_key, descendant_key, depth
INTO dim_employee_bridge
FROM h;

-- Total sales rolled up to every manager, whatever their depth
SELECT m.name AS manager, SUM(f.sales_amount) AS team_sales
FROM fact_sales f
JOIN dim_employee_bridge b ON b.descendant_key = f.employee_key
JOIN dim_employee m ON m.employee_key = b.ancestor_key
GROUP BY m.name;
```

Because a manager's own sales appear at depth 0, "my sales plus team" is automatic. Be careful in the **grand total**: summing across all managers counts each fact many times (once per ancestor). BI tools must restrict to one ancestor or to one level.

For a fixed set of levels, `GROUP BY ROLLUP(country, state, city)` produces subtotals per level; NULLs in the result mean "all", and `GROUPING()` distinguishes real NULLs from subtotal NULLs. `CUBE` gives all combinations. This works only when levels are columns, so ragged data must be padded first: a country with no state gets `state = country` (or a placeholder like "(n/a)").

### 5. Under the hood
Adjacency lists store one edge per row, and the optimizer evaluates recursive CTEs iteratively: each iteration joins the previous result to the table, with a default recursion limit in SQL Server (`MAXRECURSION`, 100). Depth *d* means *d* index seeks per starting node, and the plan cannot parallelise across iterations well.

The closure table converts recursion to one-time precomputation: rows ≈ nodes × average depth. For a 100,000-node tree with average depth 6, about 600,000 rows, trivial to join. The cost shifts to maintenance: moving a subtree requires deleting and re-inserting its pairs. Because queries become ordinary star joins, optimizers estimate them well and BI tools can use them without recursion.

Padding keeps the dimension a single row per leaf, preserving the one-join star pattern, but it hard-codes the maximum depth and the repeated names make level labels meaningless unless a `level_number` or `is_actual_level` flag is stored.

### 6. Performance and design implications
- Flat padded columns give the fastest queries and the broadest tool compatibility; use when depth ≤ ~8 and rarely changes.
- Bridge tables multiply fact rows in a join: always aggregate with a restricted ancestor, never sum the whole result.
- When the hierarchy changes over time (reorganisations), you need SCD-style versioning on the hierarchy, usually by keeping `effective_from/to` on bridge rows.
- Measure query cost with the actual plan: look for recursive spools (adjacency list) versus plain joins (bridge).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WITH cte AS (… UNION ALL …)` | recursive hierarchy walk | exploring or building closure tables |
| `OPTION (MAXRECURSION n)` | sets recursion limit (0 = unlimited) | deep trees |
| `GROUP BY ROLLUP(a,b,c)` | subtotals per level, hierarchical | flat-level reporting |
| `GROUPING(col)` / `GROUPING_ID` | tells subtotal NULL from data NULL | labelling rollup rows |
| `hierarchyid` (SQL Server) | built-in path type with ancestor tests | OLTP trees, moderate size |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Padded level columns: fixed, known depth; BI tool wants a simple star | Depth is unpredictable or changes often |
| Bridge/closure table: unlimited depth, many rollup questions | Team lacks discipline around double-counting in totals |
| Adjacency list + CTE: small trees, ad-hoc queries | Large trees queried by BI tools in many ways |
| Materialized path: subtree queries and sorting | Frequent re-parenting of large branches |

### 9. Common mistakes
- Leaving NULLs in skipped levels → groups disappear or `NULL` appears as a category → pad with the member's own value or a placeholder.
- Summing a whole bridge join → every fact counted once per ancestor → restrict to one ancestor or level.
- Hard-coding depth 5 when the org grows to 7 → truncated paths → store a depth column and monitor max depth.
- Ignoring history of reorganisations → past sales rolled to today's manager → version hierarchy edges with effective dates.
- Confusing `ROLLUP` NULLs with missing data → wrong labels → use `GROUPING()`.

### 10. 🎯 Interview kit
**30-second answer:** A ragged hierarchy has branches of different depth, so fixed level columns don't fit. I either pad the missing levels by repeating the member, or model parent-child with a bridge (closure) table so any rollup is a join and group-by.

**Structure a longer answer:** define balanced vs ragged → options (pad, adjacency + recursion, path, bridge) → example with an org chart → double-counting caveat → versioning of hierarchy changes.

**Likely follow-ups:**
- *How do you avoid double counting with a bridge?* → filter to a single ancestor/level, or use depth/top flags.
- *Which would you choose for a BI tool?* → padded columns for simplicity; bridge when depth unbounded.
- *How do you track reorganisations?* → effective-dated hierarchy rows (SCD Type 2 on the relationship).
- *What does `ROLLUP` give you?* → subtotals along a fixed column hierarchy.

**Signals interviewers look for:** you know the bridge pattern by name, you mention the double counting trap, and you think about time.

### 11. Check yourself
- [ ] Draw a ragged org chart and show its padded level-column representation.
- [ ] Write a recursive CTE that builds an ancestor-descendant closure table.
- [ ] Explain why summing the grand total over a bridge join is wrong.
- [ ] Use `ROLLUP` and `GROUPING()` for a three-level geography.
- [ ] Describe how to model a department reorganisation without rewriting history.
<!-- explainer:end -->

## Practice questions

- [Basic] [What do drill-down and roll-up mean when browsing a dimensional hierarchy?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12515_What_do_drill-down_and_roll-up_mean_when_browsing_a_dimensio.md>)
- [Intermediate] [What is drill-down and roll-up in the context of dimensional hierarchies?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12193_What_is_drill-down_and_roll-up_in_the_context_of_dimensional.md>)
- [Deep] [What is a ragged or unbalanced hierarchy, how have you modeled one, and what problems does it cause for drill-down queries?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12274_What_is_a_ragged_or_unbalanced_hierarchy_how_have_you_modele.md>)
- [Deep] [A ragged or unbalanced product hierarchy breaks your drill-down because branches vary in depth; how have you modeled one so rollups stay uniform?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12609_A_ragged_or_unbalanced_product_hierarchy_breaks_your_drill-d.md>)
- [Deep] [Walk me through modeling a hierarchical org dimension of arbitrary depth so cost rollups stay correct as reorgs happen mid-period across the enterprise.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12391_Walk_me_through_modeling_a_hierarchical_org_dimension_of_arb.md>)
- [Deep] [What is your approach to modeling ragged and unbalanced hierarchies so drill-down works uniformly across product taxonomies that vary in depth?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Q-12430_What_is_your_approach_to_modeling_ragged_and_unbalanced_hier.md>)

## Related concepts

- [Data Vault](<../Modeling_Methodologies/Data_Vault.md>) *(Modeling Methodologies)*
- [Denormalization Trade-offs and Patterns](<../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>) *(Normalization)*
- [Late Arriving Dimensions](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>) *(DBMS Concepts)*

## References

- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › CUBE and ROLLUP

