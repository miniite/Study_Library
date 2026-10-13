---
id: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Atomicity_and_Money_Transfers"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions"
tools: []
---

# Atomicity and Money Transfers

**L4 Advanced** · [↑ ACID Transactions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Atomicity makes a multi-step change all-or-nothing, and the money transfer is the textbook case, but a correct transfer also needs the right isolation, balance check and idempotency.

### 1. The big picture
Moving 100 from account A to account B is two writes: debit A, credit B. If the system fails between them, money has vanished or been created. A transaction groups the two writes so the database either applies both or neither. This is atomicity, the "A" in ACID.

Atomicity alone is not enough for a safe transfer. Two concurrent transfers can both read the same balance and both pass the overdraft check (a race, controlled by isolation). A retried request can pay twice (needs idempotency). This lesson builds a transfer step by step, showing which property guards which failure.

### 2. Core concepts and vocabulary
- **Atomic transaction** — changes committed as a unit; failure undoes all of them.
- **Rollback** — undoing the changes of an uncommitted transaction using the log.
- **Partial failure** — one step succeeded, a later one failed.
- **Lost update / write skew** — concurrent transactions overwrite or invalidate each other's assumptions.
- **Pessimistic locking** — lock the rows you read (`UPDLOCK`) so others wait.
- **Optimistic check** — write conditionally (`WHERE Balance >= @amt`) and test rows affected.
- **Idempotency key** — a client-supplied unique id that makes a retried request a no-op.
- **Ledger** — append-only record of money movements; balances derived or reconciled from it.
- **Compensating transaction** — a new transaction that semantically reverses an earlier committed one (sagas).

### 3. How it works, step by step
A safe transfer:

1. Start an explicit transaction (and `SET XACT_ABORT ON` so errors abort it).
2. Take protection against the race: either read the source row with `UPDLOCK` or do a conditional debit.
3. Debit the source only if funds suffice; check `@@ROWCOUNT`.
4. Credit the destination.
5. Insert a ledger row (with the idempotency key under a unique constraint).
6. Commit. On any error, roll back, and the engine undoes steps 2-5 from the log.
7. Application: on deadlock or serialization failure, retry the whole transaction; on duplicate key for the idempotency key, return the earlier result.

If the application crashes between the debit and credit (step 3 and 4), the connection drops; the engine rolls back the open transaction, so neither account changes. No code is needed for this beyond having started the transaction.

### 4. Worked example
```sql
CREATE TABLE Account (
    AccountID int PRIMARY KEY,
    Balance   decimal(12,2) NOT NULL CHECK (Balance >= 0)
);
CREATE TABLE TransferLog (
    RequestID uniqueidentifier PRIMARY KEY,   -- idempotency key
    FromAcct int, ToAcct int, Amount decimal(12,2),
    CreatedAt datetime2 DEFAULT sysutcdatetime()
);

CREATE PROCEDURE dbo.Transfer
    @RequestID uniqueidentifier, @From int, @To int, @Amt decimal(12,2)
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;
    BEGIN TRY
        BEGIN TRAN;
            INSERT TransferLog(RequestID, FromAcct, ToAcct, Amount)
            VALUES (@RequestID, @From, @To, @Amt);   -- duplicate = already processed

            UPDATE Account SET Balance = Balance - @Amt
             WHERE AccountID = @From AND Balance >= @Amt;
            IF @@ROWCOUNT = 0 THROW 50001, 'Insufficient funds', 1;

            UPDATE Account SET Balance = Balance + @Amt WHERE AccountID = @To;
        COMMIT;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK;
        THROW;
    END CATCH
END;
```

Reading it: the conditional `UPDATE ... AND Balance >= @Amt` fuses the check and the write into one atomic statement, so two concurrent transfers cannot both pass the check on a stale value; the second sees the reduced balance (it waits on the row lock) and fails. The `CHECK (Balance >= 0)` is a second line of defense. A retried webhook with the same `RequestID` fails on the primary key, so money moves once.

### 5. Under the hood
*(optional)* Rollback works because every modification wrote an undo-capable log record. On rollback the engine walks the transaction's log chain backwards, applies the inverse of each change, and writes compensation log records, so even a crash during rollback can be finished at recovery.

Under READ COMMITTED, a plain `SELECT Balance` followed by `UPDATE ... SET Balance = @new` can interleave: T1 reads 100, T2 reads 100, both write, one update is lost. The fixes are higher isolation (`REPEATABLE READ` / `SERIALIZABLE` hold read locks, which can deadlock), `UPDLOCK` hints, `SNAPSHOT` with update-conflict detection, or the conditional single-statement update shown above.

### 6. Performance and design implications
- Keep the transaction short: no user interaction or remote calls between `BEGIN` and `COMMIT`.
- Lock accounts in a consistent order (lowest id first) so two opposite transfers (A→B and B→A) do not deadlock; retry if they still do.
- Hot accounts (a merchant account) become a serialization point; consider per-transaction ledger inserts and periodic balance rollups.
- A ledger-first design (insert-only) scales writes better and gives audit; balances are sums or maintained snapshots.
- Cross-database or cross-service transfers cannot use one local transaction; use outbox + saga or a distributed transaction coordinator, accepting its cost.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BEGIN TRAN / COMMIT / ROLLBACK` | Define the atomic unit | Always for multi-step writes |
| `SET XACT_ABORT ON` | Errors roll back the whole transaction | Procedures with transactions |
| `TRY ... CATCH` + `THROW` | Handle and rethrow errors | Rollback then surface the error |
| `@@ROWCOUNT` | Rows affected by last statement | Detecting failed conditional update |
| `WITH (UPDLOCK, ROWLOCK)` | Lock the row on read | Read-modify-write cycles |
| `@@TRANCOUNT` | Open transaction nesting level | Safe rollback in CATCH |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Conditional update (`WHERE Balance >= x`) | Rule spans multiple rows (use locks or serializable) |
| `UPDLOCK` read then write | Many hot rows → blocking; prefer conditional update |
| `SERIALIZABLE` | Throughput matters; range locks cause deadlocks |
| Snapshot isolation + retry | Write skew is possible across different rows |
| Idempotency key table | Keys never purged → table growth |
| Saga/compensation | One database can do it with a local transaction |

### 9. Common mistakes
- Debit and credit in separate autocommit statements → crash between them loses money → one explicit transaction.
- Check balance with a plain `SELECT` then write → two transfers overdraw under READ COMMITTED → conditional update or `UPDLOCK`.
- No `XACT_ABORT` and no error handling → transaction left open after an error → `XACT_ABORT ON` and `CATCH` with rollback.
- Retrying a non-idempotent payment → double charge → idempotency key with a unique constraint.
- Holding the transaction open while calling an external API → long locks → call before/after, record state in the database.
- Swallowing the error after rollback → caller thinks it succeeded → rethrow.

### 10. 🎯 Interview kit
**30-second answer:** A transfer is a debit and a credit that must both happen or neither. I wrap them in one transaction so a failure rolls both back, make the debit conditional on sufficient balance so concurrent transfers cannot overdraw, and use an idempotency key so retries do not double-charge.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
State atomicity, show the transaction, name the race under READ COMMITTED and the fixes, then discuss deadlock ordering and idempotency.

**Likely follow-ups:**
- *App crashes after debit?* → connection drops, engine rolls back, no change.
- *Why can two transactions both pass the balance check?* → they both read the committed balance before either writes; fix with locks, higher isolation or a conditional update.
- *Idempotent payment capture?* → unique idempotency key, store result, return it on retry.
- *Why rollback in a stored procedure?* → leave no half-done work and release locks.
- *Money across services?* → saga with compensation, outbox for reliable messaging.

**Signals interviewers look for:** you mention isolation and idempotency without being asked, not just atomicity.

### 11. Check yourself
- [ ] Write a transfer with correct error handling.
- [ ] Explain the overdraft race and give two fixes.
- [ ] Show how an idempotency key prevents a double charge.
- [ ] Say how consistent lock ordering avoids deadlocks.
- [ ] Describe what the engine does on rollback.
<!-- explainer:end -->

## Practice questions

- [Basic] [Why is a bank transfer a classic example that requires atomicity?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00313_Why_is_a_bank_transfer_a_classic_example_that_requires_atomi.md>)
- [Basic] [In a money-transfer transaction why must both the debit and credit succeed or both fail?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00464_In_a_money-transfer_transaction_why_must_both_the_debit_and_.md>)
- [Basic] [Give a real-world example where a transaction is necessary such as a money transfer between two accounts.](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00501_Give_a_real-world_example_where_a_transaction_is_necessary_s.md>)
- [Basic] [How would you explain ACID to a non-technical stakeholder using a simple analogy?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00505_How_would_you_explain_ACID_to_a_non-technical_stakeholder_us.md>)
- [Intermediate] [How would you write a transaction to transfer money from account A to account B safely?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00463_How_would_you_write_a_transaction_to_transfer_money_from_acc.md>)
- [Intermediate] [What happens if the application crashes after the debit but before the credit in a transfer?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00465_What_happens_if_the_application_crashes_after_the_debit_but_.md>)
- [Deep] [Why can two transactions both pass a balance check and still overdraw an account under READ COMMITTED, and how do you close that hole?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00321_Why_can_two_transactions_both_pass_a_balance_check_and_still.md>)
- [Deep] [How would you design idempotent payment capture so a retried webhook never double-charges a customer?](<../../../PRACTICE/01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Q-00362_How_would_you_design_idempotent_payment_capture_so_a_retried.md>)

## Related concepts

- [Payment Reconciliation](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation.md>) *(System Design Core)*
- [Distributed Transactions and 2PC](<../../Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>) *(DBMS Concepts)*
- [Payment Ledgers](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Payment_Ledgers.md>) *(High Level Design)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Atomicity
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Real-Life Analogy
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Atomic transactions

