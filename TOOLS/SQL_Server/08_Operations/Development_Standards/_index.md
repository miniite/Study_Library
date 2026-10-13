---
id: "TOOLS/SQL_Server/08_Operations/Development_Standards"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Development Standards and Mistakes

**L4 Advanced** · [↑ Operations and Administration](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Development standards (naming, formatting, technical coding rules, and automated tests) make a SQL Server estate self-explanatory, so problems are found and fixed in minutes instead of being lost to decoding what things are and what they do.

### 1. The big picture
Imagine being asked to fix a slow application. The server is named `SQL01`, the database `DB03`, and the procedure `proc01` contains one unformatted line joining `tbl01`, `tbl02` and a CTE called `t3` with columns `col1`, `col2`. Half an hour goes to finding the right database, another quarter-hour to reformat the code, and only then can the investigation start. That's the cost of having no standards.

Standards cover three areas:
1. **Naming conventions**: servers, instances, databases, schemas, tables, columns, constraints, indexes, procedures.
2. **Coding standards**: *stylistic* (formatting, aliasing, comments) and *technical* (error handling, set-based code, no `SELECT *`, explicit column lists, transaction rules).
3. **Quality practices**: unit testing (tSQLt), code review, source control and CI/CD so standards are enforced, not just written down.

### 2. Core concepts and vocabulary
- **Naming convention** — a rule for how objects are named so a name states what the object is.
- **Coding standard** — rules for how code is written and structured.
- **PascalCase** — `CustomerOrders`: capitalized words, no separators; commonly used for schema objects.
- **snake_case** — `order_id`: lowercase with underscores; commonly used for variables in this book's recommendations.
- **Alias** — a name given to a column or table inside a query.
- **Computed column / persisted computed column** — a column defined by an expression; *persisted* means the value is stored so it isn't recalculated on every read.
- **Unit test (tSQLt)** — a test for one stored procedure/function in isolation, run inside SQL Server.
- **Mocking** — replacing dependencies with fakes (`FakeTable`, `SpyProcedure`) so tests are isolated from real data.
- **TDD (test-driven development)** — write the failing test first, then the code.
- **CI/CD** — automated build/test/deploy pipeline (Azure DevOps, GitHub Actions).

### 3. How it works, step by step
**Naming conventions** (recommended set)

| Object | Convention | Example |
|---|---|---|
| Schema objects | PascalCase, meaningful, no spaces/special chars | `CustomerOrders`, `InvoiceDetails` |
| Variables | snake_case/lowercase | `@order_id`, `@max_value` |
| Tables | Singular entity name, grouped by schema | `Sales.Customer`, `HR.Employee` |
| Columns | Descriptive; key with an `ID` suffix | `FirstName`, `OrderDate`, `CustomerID` |
| Stored procedures | `usp_` prefix; **never** `sp_` | `usp_GetCustomerByID` |
| Primary key | `PK_Table` | `PK_Customer` |
| Foreign key | `FK_Table_ReferencedTable` | `FK_Order_Customer` |
| Check constraint | `CHK_Table_Column` | `CHK_Order_Quantity` |
| Index | `IX_Table_Column` | `IX_Customer_LastName` |
| Servers/instances/databases | State the app or purpose | `sql-prod-timechewer`, `TimeChewer` (not `DB03`) |

Why not `sp_`? Names starting with `sp_` make SQL Server look in `master` first for the procedure, adding lookups and risking clashes with system procedures.

**Coding standards**
1. Consistent formatting: one clause per line, upper-case keywords, indentation, one column per line in a select list.
2. Meaningful aliases: `Salary * 0.10 AS Bonus`, not `AS x`; avoid spaces (`TotalSalesAmount`) in development code.
3. Comment the *why*; header block with purpose/author/change history.
4. Technical rules: explicit column lists, schema-qualified names, no `SELECT *` in production code, set-based over row-by-row, `TRY…CATCH` with `THROW`, `SET NOCOUNT ON`, parameterize.
5. Persist computed columns that are queried frequently, particularly those with costly expressions (`CASE` with many branches).

**Testing and delivery**
1. Install tSQLt into a *test* database (needs CLR enabled).
2. Create a test class per module (`TestCustomers`) and tests named `test_<Procedure>_<Condition>`.
3. Isolate with `tSQLt.FakeTable` and `tSQLt.SpyProcedure`; assert with `AssertEquals`, `AssertEqualsTable`.
4. Run suites in CI on every commit; failing tests block the merge.

### 4. Worked example
Before:

```sql
ALTER PROCEDURE dbo.proc01 AS BEGIN
;with t3 as (select col1,col2,col3 from tbl03) select t1.col1,t2.col2,t1.col2,t1.col3,t3.col1,t3.col2 from dbo.tbl01 t1 inner join tbl02 t2 on t1.col1=t2.col1 and t1.col3<55 inner join t3 on t3.col1=t1.col1 union select col1,col2,col3,NULL,NULL,'0' from dbo.tbl04;
END
```

After applying standards (names are illustrative):

```sql
CREATE OR ALTER PROCEDURE Sales.usp_GetOpenOrderSummary
AS
BEGIN
    SET NOCOUNT ON;

    -- Open orders under the credit threshold, joined to customer and region
    WITH Region AS
    (
        SELECT RegionID, RegionName, CountryCode
        FROM   Sales.Region
    )
    SELECT  o.OrderID, c.CustomerName, o.OrderDate, o.CreditLimit,
            r.RegionName, r.CountryCode
    FROM    Sales.[Order]    AS o
    JOIN    Sales.Customer   AS c ON c.CustomerID = o.CustomerID
                                  AND o.CreditLimit < 55
    JOIN    Region           AS r ON r.RegionID   = c.RegionID
    UNION
    SELECT  a.OrderID, a.CustomerName, a.OrderDate, NULL, NULL, '0'
    FROM    Sales.ArchivedOrder AS a;
END;
```

A unit test for it with tSQLt:

```sql
EXEC tSQLt.NewTestClass 'TestOrders';
GO
CREATE PROCEDURE TestOrders.[test usp_GetOpenOrderSummary excludes orders over limit]
AS
BEGIN
    EXEC tSQLt.FakeTable 'Sales.Order';
    EXEC tSQLt.FakeTable 'Sales.Customer';
    EXEC tSQLt.FakeTable 'Sales.Region';
    EXEC tSQLt.FakeTable 'Sales.ArchivedOrder';

    INSERT Sales.Customer (CustomerID, CustomerName, RegionID) VALUES (1,'A',1);
    INSERT Sales.Region   (RegionID, RegionName, CountryCode) VALUES (1,'North','GB');
    INSERT Sales.[Order]  (OrderID, CustomerID, OrderDate, CreditLimit)
    VALUES (10,1,'2025-01-01',40), (11,1,'2025-01-02',90);

    SELECT OrderID INTO #actual FROM (SELECT 0 AS OrderID WHERE 1=0) x;  -- placeholder shape
    INSERT #actual EXEC Sales.usp_GetOpenOrderSummary;                    -- adjust to proc's result shape

    EXEC tSQLt.AssertEquals @Expected = 1, @Actual = (SELECT COUNT(*) FROM #actual WHERE OrderID = 10);
END;
GO
EXEC tSQLt.Run 'TestOrders';
```

(The shape of the result set has to match the procedure; the point is the pattern: fake tables, arrange, act, assert.)

### 5. Under the hood
- **Why naming affects operations.** Names are the only documentation that's always present. Consistent constraint and index names make plan analysis, DMV output and error messages readable (`FK_Order_Customer` tells you the failing relationship).
- **`sp_` lookup.** For a procedure called `sp_x` in a user database, the engine checks `master` first, then the user database, which costs extra lookup and can pick the wrong object.
- **Persisted computed columns** are stored in the row, can be indexed, and are updated whenever dependencies change; non-persisted ones are evaluated at read time (and can only be indexed if deterministic and precise).
- **tSQLt mechanics.** Each test runs inside a transaction that is rolled back; `FakeTable` swaps the real table for an empty, constraint-free copy with the same name, which is why you must re-add constraints (`ApplyConstraint`) if the logic depends on them. It uses SQL CLR, so CLR must be enabled and assembly permissions granted; reinstall errors ("objects already exist") are fixed by recreating the test database.

### 6. Performance and design implications
- **Readability is performance work too**: tuning needs people to *see* what a query does; consistent style speeds code review.
- **Standards lead to fewer performance bugs**: explicit columns (allow covering indexes), schema-qualified names (plan reuse), no `SELECT *`, no functions on filtered columns.
- **Persist computed columns** that appear in many queries or predicates to avoid repeated calculation and allow indexing.
- **Tests as a safety net** let you refactor or tune code (change indexes, rewrite joins) with confidence that results are unchanged.
- **Cost**: enforcement tooling (linters, review checklists, CI time) must be lightweight, or teams will bypass it.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXEC sp_configure 'clr enabled',1; RECONFIGURE;` | Enables CLR | Prerequisite for tSQLt |
| `tSQLt.NewTestClass` | Creates a test schema | One per module |
| `tSQLt.FakeTable`, `ApplyConstraint` | Replace a table, optionally restore constraints | Isolate and keep realism |
| `tSQLt.SpyProcedure` | Replace a procedure with a recorder | Verify calls, stub results |
| `tSQLt.AssertEquals / AssertEqualsTable / ExpectException` | Assertions | Check results/errors |
| `tSQLt.Run` / `RunAll` | Execute tests | Locally and in CI |
| `sp_rename` | Rename objects | Careful: breaks dependencies; script via migrations |
| Azure DevOps / GitHub Actions pipeline | Build, test, deploy | Automated gates |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Strict naming conventions | Retrofitting names on a live system without a migration plan |
| Singular table names | Mixed conventions inside one database; pick one and document |
| Persisted computed columns | Rarely used columns on very large, frequently updated tables (storage + write cost) |
| tSQLt unit tests with fakes | Over-mocking: tests no longer reflect production behavior |
| Strict formatting enforced by tooling | Style debates that block delivery; agree once |

### 9. Common mistakes
- **Meaningless names** (`DB03`, `col1`) → minutes lost per incident → name by purpose.
- **`sp_` procedure prefix** → extra lookups, clashes → use `usp_` or a domain prefix.
- **Vague aliases** (`AS x`) → unreadable code → use descriptive aliases.
- **Recomputing complex expressions everywhere** → CPU waste and inconsistency → persisted computed column or view.
- **Writing tests after the code** → low coverage, tests mirror the implementation → TDD mindset.
- **Over-mocking / forgotten constraints / leaking fake tables** → false confidence → use `ApplyConstraint`, keep tests small and isolated.
- **Ignoring edge cases** (nulls, empty tables, invalid inputs) → production bugs → test them explicitly with `ExpectException` etc.

### 10. 🎯 Interview kit
**30-second answer:** Development standards are agreed rules for naming, formatting and technical coding practices, backed by automated tests and CI. They make systems self-documenting, reduce defects and speed diagnosis. Concrete examples are PascalCase objects, `PK_/FK_/IX_` names, `usp_` procedures, explicit column lists, `TRY…CATCH`, and tSQLt unit tests in the pipeline.

**Structure a longer answer:** why (cost of decoding) → naming → style → technical rules → testing/CI → how to enforce (reviews, linting, pipeline) → close with a story of a diagnosis made faster.

**Likely follow-ups:**
- *Why avoid `sp_`?* → System-procedure lookup in `master`, risk of clashes.
- *Singular or plural table names?* → Singular is often preferred (a row is an entity); the key is consistency.
- *How do you unit-test a stored procedure?* → tSQLt: fake tables, call proc, assert; run in CI.
- *When persist a computed column?* → Frequently used or expensive expressions; also to index them.
- *Risks of mocking?* → Unrealistic tests; re-apply constraints; isolate.

**Signals interviewers look for:** you treat standards as tooling plus process, know the technical reasons behind rules, and connect standards to supportability and testing.

### 11. Check yourself
- [ ] Write names for a PK, FK, check constraint and index on `Sales.Order`.
- [ ] Explain why `sp_` is discouraged for user procedures.
- [ ] Rewrite a query with vague aliases and unformatted SQL into standard style.
- [ ] Explain what `FakeTable` does and why you might call `ApplyConstraint`.
- [ ] List three over-mocking or isolation pitfalls and their fixes.
- [ ] Describe how tests run in a CI pipeline and what blocks a merge.
<!-- explainer:end -->

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Development standards › 2 Development standards
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › T-SQL Coding Standards and Conventions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Conclusion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Naming Conventions for Clarity
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Best Practices for Writing Tests
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Handling Common Installation Errors
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Common Pitfalls and Solutions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Introduction
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Common Mistakes and Avoiding Them
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Naming Conventions

