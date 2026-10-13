---
type: "practice"
difficulty: "Basic"
---

# What is a database trigger and when does it fire?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Trigger Types](<../../../../TOOLS/Oracle/PL-SQL/Trigger_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A database trigger in PL/SQL is a stored procedure that automatically executes (fires) in response to certain events or operations on a specific table or view. These triggers are essential tools for maintaining data integrity and performing actions based on specific conditions without needing to write complex procedural code every time.

To fully understand when a trigger fires, we need to look at the different types of triggers and the scenarios that initiate their execution. Let’s break it down from first principles:

#### Triage

The question is essentially asking about:

Definition: What exactly is a database trigger?
Firing Conditions: Under what circumstances does a trigger execute?

#### First-Principles Solution

Triggers in PL/SQL can be categorized into several types based on the events they respond to:

DML Triggers (Data Manipulation Language):

BEFORE: Trigger fires before an insert, update, or delete operation.
AFTER: Trigger fires after an insert, update, or delete operation.

DDL Triggers (Data Definition Language):

CREATE, ALTER, and DROP operations on tables, views, etc.

System-Triggers:

LOGON: Fires when a user logs in.
LOGOFF: Fires when a user logs out.
Detailed Explanation

Before Triggers: These are useful for validating data before it is committed to the database. For example, ensuring that a price update does not drop below a certain threshold.

SQL
CREATE OR REPLACE TRIGGER check_price
BEFORE UPDATE OF price ON product
FOR EACH ROW
BEGIN
    IF :new.price < 10 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Price cannot be less than $10');
    END IF;
END;

After Triggers: Useful for logging operations or performing additional updates that should occur after the main operation is completed.

SQL
CREATE OR REPLACE TRIGGER log_update
AFTER UPDATE OF price ON product
FOR EACH ROW
BEGIN
    INSERT INTO price_log (product_id, old_price, new_price)
    VALUES (:old.product_id, :old.price, :new.price);
END;

DDL Triggers: These are used to enforce business rules or perform administrative tasks when a schema object is altered.

SQL
CREATE OR REPLACE TRIGGER audit_table_modification
AFTER ALTER TABLE ON SCHEMA
FOR EACH STATEMENT
BEGIN
    DBMS_OUTPUT.PUT_LINE('Table was modified on ' || SYSDATE);
END;

System-Triggers:

LOGON and LOGOFF: Used for user authentication or session management.
SQL
CREATE OR REPLACE TRIGGER log_user_activity
AFTER LOGON ON SCHEMA
DECLARE
    v_username VARCHAR2(30);
BEGIN
    SELECT USERNAME INTO v_username FROM SESSION_USER;
    DBMS_OUTPUT.PUT_LINE('User ' || v_username || ' has logged in.');
END;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, triggers are powerful tools that help maintain data integrity and automate tasks. However, it's important to use them judiciously as they can impact performance if not designed carefully.

#### Common Mistakes

Firing too many triggers for simple operations, leading to performance degradation.
Not considering all edge cases (e.g., what happens during a DDL operation that might affect the trigger itself).

#### Follow-Ups

Discussing how to manage and debug triggers effectively.
Exploring other PL/SQL features like cursors and procedures.

**Comparison:**

| Triggers Type     | Description                                               |
|------------------|-----------------------------------------------------------|
| DML              | Triggered by INSERT, UPDATE, DELETE operations             |
| CREATE/DROP TABLE | Triggered when creating or dropping a table               |
| LOGON/LOGOFF      | Triggered on user login/logout                            |

**Code:**

```sql
-- Example of an AFTER UPDATE TRIGGER
CREATE OR REPLACE TRIGGER check_price
BEFORE UPDATE OF price ON product
FOR EACH ROW
BEGIN
    IF :new.price < 10 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Price cannot be less than $10');
    END IF;
END;

-- Example of an AFTER INSERT TRIGGER
CREATE OR REPLACE TRIGGER log_insert
AFTER INSERT ON product
FOR EACH ROW
BEGIN
    INSERT INTO price_log (product_id, old_price, new_price)
    VALUES (:new.product_id, NULL, :new.price);
END;
```

**Data:**

```sql
-- Sample table structure
CREATE TABLE product (
    id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    price NUMBER
);

CREATE TABLE price_log (
    id NUMBER PRIMARY KEY,
    product_id NUMBER REFERENCES product(id),
    old_price NUMBER,
    new_price NUMBER,
    log_date TIMESTAMP
);
```

**Flow:**

1. Yes
2. Yes
3. No
4. Yes
5. Yes
6. No
7. Yes
8. Yes
9. Trigger Firing
10. Is it DML?
11. BEFORE INSERT
12. AFTER INSERT
13. Is it CREATE/DROP?
14. CREATE TABLE
15. DROP TABLE
16. Is it LOGON/LOGOFF?
17. LOGON
18. LOGOFF

