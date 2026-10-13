---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a parameterized query when building the metadata filter for a vector search to stay safe from injection?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ RAG Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/RAG_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure safety in building a metadata filter for a vector search, using a parameterized query is crucial to prevent SQL injection attacks. In this context, we are dealing with user inputs that could potentially contain malicious SQL code if not properly handled. A parameterized query ensures that the data and the SQL code remain separate, thus preventing any harmful input from altering the intended SQL command.

#### Triage

The question is essentially asking how to safely handle user inputs in a SQL query when filtering metadata for a vector search. The core of the problem lies in understanding what a parameterized query is, why it’s necessary, and how to implement one effectively.

#### First-Principles Solution

Let's consider a scenario where we are building a vector search service that allows users to filter results based on metadata fields such as tags or categories. Suppose our SQL query looks something like this:

SQL
SELECT * FROM documents WHERE tag = 'example';

If an attacker can manipulate the input, they might try to inject malicious SQL, for example:

SQL
SELECT * FROM documents WHERE tag = 'example OR 1=1 -- ';

This would return all rows from the documents table. To mitigate such risks, we should use a parameterized query.

#### Implementation
in Python with SQLite

Here’s how you can implement this using Python and SQLite:

PYTHON
import sqlite3

def filter_documents(tag):
    # Establish database connection
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()

    try:
        # Parameterized query
        query = "SELECT * FROM documents WHERE tag = ?"

        # Execute the query with parameters
        cursor.execute(query, (tag,))

        # Fetch results
        results = cursor.fetchall()
        return results

    except sqlite3.Error as e:
        print(f"An error occurred: {e}")

    finally:
        conn.close()

# Example usage
filtered_documents = filter_documents('example')
print(filtered_documents)

#### Closing Summary
and Common Mistakes

In summary, using parameterized queries is essential to maintain the security of your application. By separating the SQL code from user inputs, you ensure that any potentially harmful characters are treated as literal values rather than executable commands.

#### Common mistakes

Forgetting to Use Parameters: Directly concatenating user input into the SQL query can lead to injection vulnerabilities.
Misusing Parameters: Incorrectly positioning parameters or not escaping special characters in the input data.
Not Handling Exceptions Properly: Failing to catch and handle exceptions that might arise during database operations.
Self-Check

This solution ensures that user inputs are safely handled, preventing SQL injection attacks while providing a robust metadata filter for vector searches.

