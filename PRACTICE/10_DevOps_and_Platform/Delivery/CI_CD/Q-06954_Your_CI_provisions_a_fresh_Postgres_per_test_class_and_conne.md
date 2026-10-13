---
type: "practice"
difficulty: "Intermediate"
---

# Your CI provisions a fresh Postgres per test class and connection setup dominates runtime; walk me through pooling, template databases, and transaction rollback isolation as ways to cut that cost.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Integration Testing and Test Doubles](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Integration_Testing_and_Test_Doubles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up Continuous Integration (CI) for backend testing with PostgreSQL, one of the biggest challenges is ensuring efficient use of resources while maintaining reliability and consistency. In your scenario, where a fresh Postgres instance is provisioned per test class, the overhead can become significant. Let's explore how using connection pooling, template databases, and transaction rollback isolation can help mitigate this cost.

TL;DR

To reduce resource consumption in CI/CD setups for backend tests with PostgreSQL, we can leverage connection pooling to reuse database connections, utilize template databases for shared resources, and ensure proper transaction handling through auto-commit settings or explicit management. These strategies not only save computational power but also enhance the overall efficiency of your testing environment.

#### Triage

The core question revolves around optimizing resource usage in a CI/CD pipeline where multiple test classes are run independently with their own PostgreSQL databases. The primary focus is on reducing overhead related to database connection setup and teardown, as well as ensuring efficient transaction management during tests.

#### Assumptions
Multiple Test Classes: Each test class runs in isolation, meaning it gets its own database instance.
PostgreSQL Environment: The use of PostgreSQL databases for testing.
Connection Overhead: Significant time is spent setting up and tearing down connections per test run.

#### Solution
Connection Pooling

Connection pooling allows multiple applications to share a common pool of established database connections, rather than opening and closing new connections on each request. This approach significantly reduces the overhead associated with establishing a connection every time a test starts.

#### Example
Setup:

BASH
# PostgreSQL Configuration for Connection Pooling
max_connections = 100 # Adjust based on your needs
shared_buffers = 128MB # Adjust based on available memory and workload

# pg_hba.conf
host    all             all             127.0.0.1/32            trust

Code Example:

PYTHON
import psycopg2
from contextlib import closing

# Connection Pool Configuration
pool_size = 10
connection_pool = []

def get_db_connection():
    conn = psycopg2.connect(database="template1")
    connection_pool.append(conn)
    return conn

def close_all_connections():
    for conn in connection_pool:
        conn.close()
Template Databases

Template databases provide a way to share common schema and data across multiple test environments without duplicating the entire database setup. By using template databases, you can create new instances quickly with minimal overhead.

#### Example
Setup:

Create a base template database.
Use this template to create test databases.
SQL
# Base Template Database (template1)
CREATE DATABASE template1 WITH TEMPLATE = "base_template";

# Test Database Creation from Template
CREATE DATABASE test_db TEMPLATE = template1;
Transaction Rollback Isolation

Ensuring that transactions are properly rolled back after tests helps maintain a clean state between test runs. This can be managed through auto-commit settings or explicit transaction management.

#### Example
Setup:

PYTHON
# Ensuring Auto-Commit is Off for Explicit Transactions
def setup_test_db():
    conn = get_db_connection()
    with closing(conn.cursor()) as cursor:
        # Turn off autocommit and start a new transaction
        cursor.execute("SET AUTOCOMMIT TO OFF")
        # Perform test operations here
        # Use `cursor.execute('ROLLBACK')` to roll back changes if needed

# Ensure Auto-Commit is On for Clean Up
def teardown_test_db():
    conn = get_db_connection()
    with closing(conn.cursor()) as cursor:
        cursor.execute("SET AUTOCOMMIT TO ON")

#### Closing Summary

By implementing connection pooling, utilizing template databases, and ensuring proper transaction management, you can significantly reduce the overhead associated with database setup and teardown in your CI/CD pipeline. This not only optimizes resource usage but also improves the reliability and scalability of your testing environment.

#### Common Mistakes/Follow-Ups
Overlooking Connection Pool Configuration: Failing to configure connection pooling properly can lead to performance bottlenecks.
Insufficient Use of Template Databases: Not leveraging template databases for shared resources can result in redundant database setups, increasing costs and complexity.
Inadequate Transaction Management: Poor transaction handling can leave the database in an inconsistent state, leading to failed tests or data corruption.

**Comparison:**

```
tests
uses
shares
TEST_CLASS
DATABASE
using
CONNECTION_POOL
TEMPLATE_DATABASE
```

**Code:**

```python
import psycopg2
from contextlib import closing

def get_db_connection():
    conn = psycopg2.connect(database="template1")
    return conn

def close_all_connections(connection_pool):
    for conn in connection_pool:
        conn.close()

connection_pool = []

def setup_test_db():
    conn = get_db_connection()
    with closing(conn.cursor()) as cursor:
        # Turn off autocommit and start a new transaction
        cursor.execute("SET AUTOCOMMIT TO OFF")
        # Perform test operations here

def teardown_test_db():
    conn = get_db_connection()
    with closing(conn.cursor()) as cursor:
        cursor.execute("SET AUTOCOMMIT TO ON")
```

**Example input / output:**

```text
# Example Input/Output for Setup and Teardown

def setup_teardown_example():
    # Setup Phase
    connection_pool = [get_db_connection()]
    setup_test_db()

    # Test Phase (Mock)
    print("Test operations performed here...")

    # Teardown Phase
    teardown_test_db()
    close_all_connections(connection_pool)

setup_teardown_example()
```

**Data:**

```sql
-- Base Template Database (template1)
CREATE DATABASE template1 WITH TEMPLATE = "base_template";

-- Test Database Creation from Template
CREATE DATABASE test_db TEMPLATE = template1;
```

**Flow:**

1. Setup Test DB
2. Open Connection
3. Create Transaction
4. Test Operations
5. Rollback/Commit
6. Tear Down Test DB
7. Close Connections

