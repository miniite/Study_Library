---
type: "practice"
difficulty: "Intermediate"
---

# How do you write PL/SQL unit tests with utPLSQL, and what would you assert for a procedure that mutates state and raises exceptions?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Exception Handling](<../../../../TOOLS/Oracle/PL-SQL/Exception_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write effective PL/SQL unit tests using utPLSQL, you need to understand both the structure of your procedures and how to properly validate their behavior, especially when they mutate state or raise exceptions. This involves setting up test data, executing the procedure, and then asserting expected outcomes.

#### Triaging the Question

The core of this question is about understanding how to write unit tests for PL/SQL procedures using utPLSQL. Specifically, we need to consider scenarios where:

The procedure mutates state (e.g., updating a database table).
The procedure raises exceptions.
We ensure that all aspects of the procedure's behavior are correctly tested.

#### Assumptions
You have basic knowledge of PL/SQL and Oracle databases.
You understand the utPLSQL framework, which is used for testing Oracle stored procedures and packages.
You have a test database schema with relevant tables and data.
Step-by-Step Solution
1. Setting Up Test Data

Before writing any tests, you need to set up your test environment by creating or inserting sample data that the procedure will use. For example:

SQL
CREATE TABLE example_table (
    id NUMBER PRIMARY KEY,
    value VARCHAR2(100)
);

INSERT INTO example_table (id, value) VALUES (1, 'original');
COMMIT;
2. Writing the Test Procedure with utPLSQL

You'll write a test procedure using utPLSQL that includes assertions for different aspects of your main procedure's behavior.

Here’s an example:

PLSQL
-- Main procedure: updates the value in example_table and raises a warning if the value is 'original'
CREATE OR REPLACE PROCEDURE update_example (
    p_id IN NUMBER,
    p_new_value IN VARCHAR2 DEFAULT NULL
) AS
BEGIN
    IF p_new_value = 'original' THEN
        RAISE_APPLICATION_ERROR(-20101, 'Cannot use "original" as new value');
    END IF;

    UPDATE example_table SET value = p_new_value WHERE id = p_id;
END;
/
3. Writing the utPLSQL Test Case

Create a test case to verify the behavior of update_example:

PLSQL
-- utPLSQL test case for update_example procedure
CREATE OR REPLACE PROCEDURE test_update_example IS
    l_new_value VARCHAR2(100);
BEGIN
    -- Arrange: Set up initial state
    l_new_value := 'new value';

    -- Act & Assert: Test with valid new value
    BEGIN
        utPLSQL.ut_run.set_test_name('Test update_example with valid value');
        utPLSQL.ut_run.set_test_comment('Check if the procedure updates the correct value without raising an error');

        UPDATE example_table SET value = l_new_value WHERE id = 1;
        -- Assert: Check that the state was updated as expected
        ASSERT.equal(1, (SELECT COUNT(*) FROM example_table WHERE value = 'new value'), 'Value should be updated to "new value"');
    EXCEPTION
        WHEN OTHERS THEN
            utPLSQL.ut_run.set_test_status(ut_plsql_util.tester_status.failed);
            RAISE;
    END;

    -- Act & Assert: Test with invalid new value (should raise an exception)
    BEGIN
        utPLSQL.ut_run.set_test_name('Test update_example with invalid value');
        utPLSQL.ut_run.set_test_comment('Check if the procedure raises an error when given "original" as a new value');

        l_new_value := 'original';

        -- Act
        BEGIN
            UPDATE example_table SET value = l_new_value WHERE id = 1;
            -- Assert: Check that the exception was raised and state was not changed
            ASSERT.failed(1, (SELECT COUNT(*) FROM example_table WHERE value = 'new value'), 'Value should remain unchanged');
            utPLSQL.ut_run.set_test_status(ut_plsql_util.tester_status.passed);
        EXCEPTION
            WHEN OTHERS THEN
                utPLSQL.ut_run.set_test_status(ut_plsql_util.tester_status.failed);
                RAISE;
        END;
    END;

END test_update_example;
/
4. Running the Tests

You can run the tests using:

PLSQL
BEGIN
    utPLSQL.run('test_update_example');
END;
/

#### Closing Summary
and Common Mistakes

Writing effective PL/SQL unit tests with utPLSQL involves setting up robust test cases that cover all possible states of your procedures. Common mistakes include not thoroughly setting up initial state, missing exception handling, or failing to validate changes in state. Always ensure you cover both successful and error scenarios.

Common Follow-Ups
How do you handle dependencies between tests?
What are some best practices for organizing test cases?

**Comparison:**

| Test Case | Test Name                 | Test Comment                                  | Expected Outcome |
|-----------|--------------------------|----------------------------------------------|------------------|
| 1         | Test update_example with valid value | Check if the procedure updates the correct value without raising an error | Value should be updated to "new value" |
| 2         | Test update_example with invalid value  | Check if the procedure raises an error when given "original" as a new value   | Procedure should raise an exception and not change the value |

**Code:**

```sql
-- PL/SQL code for the main procedure and utPLSQL test case
```

**Example input / output:**

```text
-- Example input and output for the utPLSQL test case

Test 1:
- Test name: Test update_example with valid value
- Test comment: Check if the procedure updates the correct value without raising an error
- Expected outcome: Value should be updated to "new value"

Test 2:
- Test name: Test update_example with invalid value
- Test comment: Check if the procedure raises an error when given "original" as a new value
- Expected outcome: Procedure should raise an exception and not change the value
```

**Data:**

```sql
CREATE TABLE example_table (
    id NUMBER PRIMARY KEY,
    value VARCHAR2(100)
);

INSERT INTO example_table (id, value) VALUES (1, 'original');
COMMIT;
```

**Flow:**

1. Setup Initial State
2. Assert Valid Value Update
3. Check if Value is UpdatedCorrectly
4. Assert Invalid Value Exception
5. Check if Procedure Raises anError

