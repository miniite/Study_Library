---
id: "TOOLS/dbt/dbt_Core/dbt_Tests"
level: 4
type: "concept"
topic: "TOOLS/dbt/dbt_Core"
tools: ["dbt"]
---

# dbt Tests

**L4 Advanced** · [↑ dbt Core](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A dbt test is a SELECT that returns the *rows that violate* an assertion; if it returns any rows the test fails — and dbt gives you four ready-made generic tests, your own singular and custom generic tests, and severity controls to make this a practical data-quality gate.

### 1. The big picture
Transformations can silently produce wrong data: duplicates after a join, nulls from a source change, orphaned foreign keys. dbt treats **data tests** as first-class: they sit next to the models, run in the same DAG with `dbt build`, and stop downstream models when a check on an upstream model fails. They are not a replacement for monitoring/observability, but they are the cheapest first line of defence and document the model's contract ("`order_id` is unique and never null").

### 2. Core concepts and vocabulary
- **Generic test** — a parameterised, reusable test written once as a macro and applied in YAML to a model/column.
- **Singular test** — a one-off SQL file in `tests/` that returns failing rows.
- **Built-in generic tests** — `unique`, `not_null`, `accepted_values`, `relationships`.
- **`severity`** — `error` (default; fails the run) or `warn`.
- **`warn_if` / `error_if`** — thresholds on number of failing rows (e.g. `error_if: ">100"`).
- **`where`** — limit a test to a subset (e.g. last 7 days).
- **`store_failures`** — save failing rows to a table for inspection.
- **Source freshness** — separate check that raw tables are recent enough.
- **Unit tests** (dbt 1.8+) — test model *logic* with mocked inputs and expected outputs, before real data runs.
- **Test packages** — `dbt_utils`, `dbt_expectations`, `elementary` provide many more assertions.

### 3. How it works, step by step
1. You declare tests in a model's YAML file (`columns:` → `tests:`/`data_tests:`) or add a SQL file under `tests/`.
2. dbt compiles each test into a query. For `not_null` on `customer_id` this is roughly: `select customer_id from model where customer_id is null`.
3. The query runs in the warehouse; dbt counts returned rows.
4. 0 rows → pass. >0 rows → fail (or warn, depending on severity/threshold).
5. With `dbt build`, a failed test (error severity) on a model causes its downstream models to be **skipped**, preventing bad data from propagating.

### 4. Worked example
```yaml
# models/marts/_marts.yml
version: 2
models:
  - name: fct_orders
    columns:
      - name: order_id
        data_tests: [unique, not_null]
      - name: status
        data_tests:
          - accepted_values:
              values: ['placed', 'shipped', 'returned', 'cancelled']
              config: {severity: warn}
      - name: customer_id
        data_tests:
          - relationships:
              to: ref('dim_customers')
              field: customer_id
              config: {where: "ordered_at >= current_date - 7", store_failures: true}
```
```sql
-- tests/assert_no_negative_revenue.sql  (singular test)
select order_id, amount
from {{ ref('fct_orders') }}
where amount < 0
```
```sql
-- macros/test_positive_values.sql  (custom generic test)
{% test positive_values(model, column_name) %}
select {{ column_name }} from {{ model }} where {{ column_name }} <= 0
{% endtest %}
```
Run: `dbt test --select fct_orders` or `dbt build --select +fct_orders`. A failing run prints the test name, number of failures, and the path to the compiled SQL; with `store_failures` the offending rows are in a `…_dbt_test__audit` schema.

### 5. Under the hood
- All tests are "count rows from a failure query"; the wrapper is effectively `select count(*) as failures, count(*) != 0 as should_warn/should_error from (<test sql>)`.
- `unique` is `group by col having count(*) > 1`; `not_null` is `where col is null`; `accepted_values` is rows not in the list; `relationships` is a left anti-join from child to parent (rows whose key has no parent). Note that `relationships` ignores nulls in the child column — pair it with `not_null` if nulls are invalid.
- `unique` and `not_null` are different checks: uniqueness allows nulls on most engines (a unique-test counts duplicate groups; null groups may or may not flag depending on SQL), so primary-key columns need **both**.
- Tests don't enforce anything in the warehouse — unlike database constraints, they detect after the fact. On warehouses that don't enforce constraints (Snowflake, BigQuery), dbt tests *are* the integrity layer.
- Test results are in `run_results.json` with `status: pass|fail|warn|error|skipped`.

### 6. Performance and design implications
- Tests scan data: `unique` on a billion-row table costs real compute. Limit with `where`, test only recent partitions of big tables, and avoid duplicating the same test across layers.
- Put the **strict tests at the boundary**: sources/staging (so you catch bad inputs early) and on marts' keys (so consumers can trust them).
- Triage a failing test at 2 a.m.: (1) severity — is it a business-critical column or a warn candidate? (2) scope — how many rows and since when (`store_failures`, query the failures)? (3) blast radius — which downstream models/dashboards (`dbt ls --select <model>+`, exposures)? (4) action — block downstream (error), publish with warning, or quarantine bad rows; (5) root cause — source change vs transformation bug; (6) add/adjust a test so it can't recur.
- Reach for **dbt-expectations/custom generics** when the rule is richer than the four built-ins: row-count within expected range, value distribution, regex format, referential rules across multiple columns, "revenue by day within ±20% of trailing average".
- Unit tests complement data tests: they catch logic bugs with deterministic fixtures even when production data is clean.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `dbt test` | run all tests | after models built |
| `dbt test --select model` / `test_type:generic` | subset | iteration |
| `dbt build` | models + tests in DAG order, blocks downstream | CI/prod |
| `severity`, `warn_if`, `error_if` | graded outcomes | noisy or new checks |
| `where` | restrict rows tested | large tables |
| `--store-failures` / `store_failures: true` | persist bad rows | debugging |
| `dbt source freshness` | raw data recency | upstream SLA |
| `{% test %}` macro | custom generic | repeated business rule |
| `tests/*.sql` | singular test | one-off rule |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Generic test for a rule that repeats across columns | the rule is unique to one model (use singular) |
| Singular test for complex multi-table logic | you'll need it in many places (promote to generic) |
| `severity: warn` for new or low-confidence checks | the failure should block publication |
| `dbt build` so tests gate downstream | you need models always published regardless of quality |
| Package tests for statistical/shape checks | simple key tests suffice, or the package cost/maintenance isn't justified |

### 9. Common mistakes
- Only `unique` on a key, no `not_null` → nulls slip through → test both.
- Relying on `relationships` for nullable FKs → nulls ignored → add `not_null` if required.
- Tests on huge tables without `where` → slow, expensive runs → scope them.
- Everything is `error` → alert fatigue and blocked pipelines → grade with severity/thresholds.
- Test passes because model is empty → add row-count/recency checks.
- Confusing generic vs singular → duplicated SQL → use generic for repeatable rules.

### 10. 🎯 Interview kit
**30-second answer:** dbt tests are queries that return violating rows; zero rows means pass. The four built-ins are `unique`, `not_null`, `accepted_values` and `relationships`; beyond that you write singular tests (SQL files) or custom generic tests (macros). Used with `dbt build`, a failing test stops downstream models.

**Structure a longer answer:** clarify what's being protected → built-ins for keys/domains/FKs → custom for business rules → severity, scoping, store_failures → CI and monitoring integration.

**Likely follow-ups:**
- *unique vs not_null?* → different failure queries; keys need both.
- *Generic vs singular?* → parameterised reusable macro vs one-off SQL in `tests/`.
- *Where does a singular test live?* → `.sql` file under the `tests/` directory, returning failing rows.
- *relationships purpose?* → referential integrity from a child column to a parent key, essential when the warehouse doesn't enforce FKs.
- *When a test fails at 2 a.m.?* → triage severity, scope, blast radius, decide block/warn, fix root cause, strengthen test.

**Signals interviewers look for:** understanding tests as detective (not preventive) controls, cost awareness, severity design, and tying quality gates to the DAG.

### 11. Check yourself
- [ ] Write the compiled SQL for each of the four built-in tests.
- [ ] Create a singular test and then convert it to a custom generic one.
- [ ] Explain how a failing test affects `dbt run` versus `dbt build`.
- [ ] Design a test suite for a `fct_orders` table including freshness and row counts.
- [ ] Describe how you would triage a failing not_null test on a critical column.
<!-- explainer:end -->

## Implementations (tools)

- [dbt](<../_index.md>)

## Practice questions

- [Basic] [Name the four built-in generic tests in dbt and what each one checks.](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22670_Name_the_four_built-in_generic_tests_in_dbt_and_what_each_on.md>)
- [Basic] [How does the unique test differ from the not_null test in dbt?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22671_How_does_the_unique_test_differ_from_the_not_null_test_in_db.md>)
- [Basic] [What is the accepted_values test used for? Give an example.](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22673_What_is_the_accepted_values_test_used_for_Give_an_example.md>)
- [Intermediate] [What does the relationships test verify and how is it useful for data integrity?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22672_What_does_the_relationships_test_verify_and_how_is_it_useful.md>)
- [Intermediate] [What is the difference between a generic test and a singular test in dbt?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22674_What_is_the_difference_between_a_generic_test_and_a_singular.md>)
- [Intermediate] [How do you write a singular test in dbt, and where does the SQL file live?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22675_How_do_you_write_a_singular_test_in_dbt_and_where_does_the_S.md>)
- [Intermediate] [A not_null test on a critical column started failing at 2am. How do you triage severity, decide whether to block downstream, and communicate to consumers?](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22790_A_not_null_test_on_a_critical_column_started_failing_at_2am_.md>)
- [Intermediate] [When would you reach for dbt-expectations or a custom generic test over the four built-ins, and give an example of a business rule you encoded as a test.](<../../../PRACTICE/TOOLS/dbt/dbt_Core/Q-22792_When_would_you_reach_for_dbt-expectations_or_a_custom_generi.md>)

## Related concepts

- [Uniqueness Checks](<../../../02_Data_Engineering/Data_Quality/Validation/Uniqueness_Checks.md>) *(Validation)*
- [Validating Loads and Deduplication](<../../../02_Data_Engineering/Data_Quality/Validation/Validating_Loads_and_Deduplication.md>) *(Validation)*
- [Data and Analytics CI Gates](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Data_and_Analytics_CI_Gates.md>) *(CI CD)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — CI/CD solutions › Other technologies
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Introduction
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Writing Your First Test
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Best Practices for Writing Tests
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Advanced Example: Testing Business Logic
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Data Quality Monitoring Strategies and the Role of Automation
- *Data Pipelines Pocket Reference* — James Densmore — Data Validation in Pipelines › Structure of a Validation Test
- *Data Pipelines Pocket Reference* — James Densmore — Data Validation in Pipelines › Validation Test Examples

