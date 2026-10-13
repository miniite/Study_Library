---
id: "TOOLS/SQL_Server/04_Query_Processing/Automatic_Tuning"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Automatic Tuning

**L4 Advanced** · [↑ Query Processing and Tuning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Automatic tuning lets the engine do two mechanical tuning jobs itself: Automatic Plan Correction (detect a plan regression in Query Store, force the last good plan, verify it helped, undo it if not) in SQL Server 2017+ and Azure SQL, and automatic index create/drop in Azure SQL Database.

### 1. The big picture
Some tuning decisions are mechanical. *"The query got slower when its plan changed, and the old plan was faster"* can be checked with data and acted on without human judgment. Microsoft automated exactly these cases on top of **Query Store** (previous lesson):
1. **Automatic Plan Correction (APC)**, also known as `FORCE_LAST_GOOD_PLAN`. Available in SQL Server 2017+ (Enterprise and Developer, and Standard from 2022) and Azure SQL. **Off by default in SQL Server, on by default in Azure SQL Database.**
2. **Automatic index management**: `CREATE_INDEX` / `DROP_INDEX`, **Azure SQL Database only.** It uses telemetry about your workload (not your data) to recommend, create, validate and remove indexes.

These are most valuable when you manage hundreds or thousands of databases and cannot look at each regression by hand. They do not replace tuning skills: they only fix *plan-choice regressions* and *simple missing or unused indexes*, not bad code, bad design or bad statistics.

### 2. Core concepts and vocabulary
- **Plan regression**: a query's performance gets worse because its plan changed (parameter sniffing, statistics update, cumulative update, compatibility change).
- **Tuning recommendation**: a row in `sys.dm_db_tuning_recommendations` describing a detected regression, the regressed and recommended plan, the estimated gain, the state and the script to apply it.
- **Last known good plan**: the most recent plan of the query that performed better than the current one.
- **Verification**: after forcing, the engine measures the query again. If it is not better, or errors/timeouts appear, it reverts.
- **Recommendation states**: `Active` (open), `Verifying` (forced, being checked), `Success`, `Reverted`, `Expired`.
- **error_prone**: recommendation detail meaning the forced plan caused errors.

### 3. How it works, step by step
**Automatic plan correction follows the same loop a DBA would:**
1. **Monitor**: Query Store collects runtime statistics per plan.
2. **Detect**: the engine notices that a query's current plan performs significantly worse (mostly CPU time) than a previous plan, with enough executions to be statistically meaningful.
3. **Recommend**: a row appears in `sys.dm_db_tuning_recommendations`, even when automatic correction is *off*. You can read it and apply the script by hand.
4. **Act** (only if `FORCE_LAST_GOOD_PLAN = ON`): the engine forces the last good plan, exactly like `sp_query_store_force_plan`.
5. **Verify**: it watches the forced plan's performance. If it is better → state `Success`. If performance doesn't improve, or the forced plan causes errors or timeouts → it **unforces** the plan (`Reverted`, possibly `error_prone`).
6. **Re-evaluate**: if the data or schema change makes the forced plan invalid, normal forcing rules apply (forcing fails and the optimizer's plan is used).

Important details:
- `sys.dm_db_tuning_recommendations` lives **in memory**: a restart clears it. Plans already forced **stay forced** (they are normal Query Store forced plans), but unfinished verification is lost.
- An automatically forced plan looks like any other forced plan in Query Store reports and DMVs (`is_forced_plan = 1`, *Use Plan* in the plan properties). You can tell them apart through `plan_forcing_type_desc = AUTO` in `sys.query_store_plan`.

**Azure SQL automatic index management:**
1. The service analyzes workload telemetry (missing-index signals, query metrics) with machine-learning models.
2. It recommends indexes to create, and unused or duplicate indexes to drop.
3. If enabled, it creates the index during low-activity periods, then **validates**: compares query performance before/after. If performance regresses, it drops the index again.
4. Indexes created this way are visible in `sys.indexes` (names start with `nci_wi_`) and listed with their history in the portal and in `sys.dm_db_tuning_recommendations`.
5. Microsoft states it uses query metrics, not your data content.

### 4. Worked example
```sql
-- Setup (MakeBigAdventure-style tables, Query Store on)
CREATE INDEX ix_ActualCost ON dbo.bigTransactionHistory (ActualCost);
GO
CREATE OR ALTER PROCEDURE dbo.ProductByCost (@ActualCost money)
AS
SELECT bth.ActualCost
FROM dbo.bigTransactionHistory AS bth
JOIN dbo.bigProduct AS p ON p.ProductID = bth.ProductID
WHERE bth.ActualCost = @ActualCost;
GO
ALTER DATABASE CURRENT SET QUERY_STORE CLEAR;  -- fresh history for the demo

-- Build a "good" history: many calls with a selective value
EXEC dbo.ProductByCost @ActualCost = 8.2205;   -- repeat a few hundred times

-- Cause a regression: evict the plan, compile with a value returning many rows
DECLARE @ph varbinary(64);
SELECT @ph = plan_handle FROM sys.dm_exec_procedure_stats
WHERE object_id = OBJECT_ID(N'dbo.ProductByCost');
DBCC FREEPROCCACHE(@ph);
EXEC dbo.ProductByCost @ActualCost = 0.0;      -- sniffed: large-row plan
EXEC dbo.ProductByCost @ActualCost = 8.2205;   -- repeat: now slower

-- Read the recommendation
SELECT ddtr.type, ddtr.reason, ddtr.state,
       JSON_VALUE(ddtr.details, '$.implementationDetails.script') AS script,
       ddtr.score, ddtr.last_refresh
FROM sys.dm_db_tuning_recommendations AS ddtr;
```
You will see a `FORCE_LAST_GOOD_PLAN` recommendation whose `reason` says something like *"Average query CPU time changed from X ms to Y ms"*, and a script such as `exec sp_query_store_force_plan @query_id = …, @plan_id = …`. You can run the script yourself, or let the engine do it:
```sql
ALTER DATABASE CURRENT SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON);

SELECT name, desired_state_desc, actual_state_desc, reason_desc
FROM sys.database_automatic_tuning_options;
```
After enabling, repeat the regression. The state moves to `Verifying` and then `Success` (or `Reverted`), and the Queries With Forced Plans report shows the plan forced.

### 5. Under the hood
- Detection is based on Query Store aggregates (CPU time per execution is the main signal) and requires enough executions on both plans to avoid acting on noise. Exact thresholds are internal and have changed between versions.
- The `details` column is JSON: `planForceDetails` (query_id, regressed and recommended plan IDs, CPU numbers), `implementationDetails` (method, script) and the estimated gain. Parse it with `JSON_VALUE` / `OPENJSON`.
- Because the DMV is in memory, collect it regularly (a scheduled job into a table) if you want history of recommendations.
- APC can only choose among plans the query **has already used**. It never invents a new plan, so it cannot fix a query that has always been slow.
- In Azure, index recommendations come from the same signals as missing-index DMVs plus workload analysis. They are validated against real query performance, which is why they are safer than blindly creating every missing-index hint.

### 6. Performance and design implications
- Enabling `FORCE_LAST_GOOD_PLAN` is low risk once Query Store is healthy. The engine verifies and reverts automatically. It is most useful for large estates and third-party applications.
- It treats symptoms. Regularly review auto-forced plans and fix root causes (parameter sensitivity, statistics, code), or forced plans pile up.
- Auto-forced plans can block better plans after you add an index. Review after schema changes.
- In Azure, auto index creation can add write overhead and storage. Review created indexes and watch DML-heavy tables.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER DATABASE … SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON)` | Enable automatic plan correction | After Query Store is configured |
| `sys.dm_db_tuning_recommendations` | Current recommendations, state, script (in memory) | Review even when auto mode is off |
| `sys.database_automatic_tuning_options` | Desired vs actual state and reason | Checking it's really on |
| `sys.query_store_plan.plan_forcing_type_desc` | MANUAL vs AUTO forcing | Distinguishing auto-forced plans |
| Azure: `ALTER DATABASE … SET AUTOMATIC_TUNING (CREATE_INDEX = ON, DROP_INDEX = ON)` | Auto index management | Azure SQL Database |
| Azure portal → Automatic tuning / Performance recommendations | UI for options and history | Azure fleets |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| APC: many databases, frequent sniffing/upgrade regressions | Expecting it to fix queries that were always slow |
| Recommendations only (APC off): you want control | Nobody reads the DMV, so recommendations are wasted |
| Azure auto index create: small team, many databases | Write-heavy tables; strict schema control |
| Azure auto index drop: cleanup of unused indexes | Indexes used only by rare (monthly) processes |

### 9. Common mistakes
- Enabling APC without Query Store running READ_WRITE → nothing happens → check `actual_state_desc` in both views.
- Assuming recommendation history persists → lost at restart → capture it into a table on a schedule.
- Letting auto-forced plans accumulate → stale forced plans block better ones → review AUTO-forced plans monthly.
- Thinking automatic tuning replaces skills → bad code and design stay unfixed → use it as a safety net.

### 10. 🎯 Interview kit
**30-second answer:** "SQL Server 2017+ can automatically correct plan regressions using Query Store data: it detects a query whose new plan is significantly slower than a previous one, forces the last good plan, verifies the improvement, and reverts if it doesn't help. It's off by default on-premises and on in Azure SQL Database. Recommendations are visible in sys.dm_db_tuning_recommendations even when it's off. Azure SQL Database can also create and drop indexes automatically, with validation. It only handles mechanical cases, so root causes still need fixing."
**Structure a longer answer:** what is automatable → APC loop (detect, force, verify, revert) → configuration and DMVs → limits → Azure index management.
**Likely follow-ups:** *What happens on restart?* → recommendations DMV cleared, forced plans remain. *Can it pick a plan never used before?* → no. *How do you tell auto-forced from manual?* → `plan_forcing_type_desc`.
**Signals interviewers look for:** knows it depends on Query Store; knows defaults per platform; understands verification and revert; sees it as a safety net, not a fix.

### 11. Check yourself
- [ ] Reproduce a regression and read the recommendation's `reason` and `script`.
- [ ] Enable `FORCE_LAST_GOOD_PLAN` and confirm `actual_state_desc`.
- [ ] Parse the `details` JSON to get the regressed and recommended plan IDs.
- [ ] Explain why APC cannot fix a query that was always slow.
- [ ] List what survives a restart and what doesn't.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Automatic Tuning in Azure SQL Database
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 22 Automated Tuning In Azure And Sql Server
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 21 Intelligent Query Processing
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Best Practices for Forcing Plans

