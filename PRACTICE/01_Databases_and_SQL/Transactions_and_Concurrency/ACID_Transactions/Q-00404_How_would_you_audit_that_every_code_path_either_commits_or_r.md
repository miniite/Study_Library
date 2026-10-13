---
type: "practice"
difficulty: "Intermediate"
---

# How would you audit that every code path either commits or rolls back its transaction, even on exceptions?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Commit Rollback and Savepoints](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Commit_Rollback_and_Savepoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure every code path in a database transaction either commits or rolls back its transaction, even on exceptions, we need to implement a robust auditing mechanism that covers all possible execution paths. This involves understanding ACID properties and leveraging programming constructs effectively.

#### Triage

The core of the question is about ensuring atomicity through proper handling of transactions—specifically committing or rolling back based on the outcome of the code path. We are looking for techniques to catch exceptions, handle them appropriately, and ensure a transaction state update that reflects whether the process was successful or not. The key assumptions include:

Transactions must be handled within a try-catch block.
Exception handling should be comprehensive to cover all possible error cases.

#### First-Principles Solution
Transaction Management: Ensure transactions are started at the beginning of your code path using BEGIN TRANSACTION in SQL or equivalent transaction management calls like DB.BeginTransaction() in ADO.NET.
Try-Catch Block: Wrap the critical section of your code with a try-catch block to handle exceptions gracefully.
Commit on Success: Within the try block, execute the database operations that should be committed upon success.
Rollback on Failure: In the catch block, ensure any uncommitted changes are rolled back and log the failure.

#### Implementation
Example

Here’s a sample implementation in C# using ADO.NET:

CSHARP
using (var connection = new SqlConnection(connectionString))
{
    try
    {
        connection.Open();

        // Start transaction
        using (var transaction = connection.BeginTransaction())
        {
            // Execute SQL commands within the transaction
            var command = new SqlCommand("UPDATE Employees SET Salary = @NewSalary WHERE EmployeeID = @EmployeeID", connection, transaction);
            command.Parameters.AddWithValue("@NewSalary", newSalary);
            command.Parameters.AddWithValue("@EmployeeID", employeeId);

            int rowsAffected = command.ExecuteNonQuery();

            // Check if the update was successful
            if (rowsAffected > 0)
            {
                // Commit the transaction on success
                transaction.Commit();

#### Cons
ole.WriteLine("Transaction committed successfully.");
            }
        }
    }
    catch (Exception ex)
    {
        // Rollback the transaction and log the exception
        connection.Rollback();

#### Cons
ole.WriteLine($"An error occurred: {ex.Message}");
    }
}

#### Closing Summary

In summary, to ensure every code path commits or rolls back a transaction, implement comprehensive try-catch blocks around your database operations. Commit transactions on successful execution and rollback on any exceptions. This approach ensures data integrity and consistency.

#### Common Mistakes
and Follow-Ups
Forgetting Rollback: Ensure that all uncommitted changes are rolled back in the catch block.
Inadequate Exception Handling: Catch blocks should be designed to handle specific exception types, not just generic Exception.
Logging Exceptions: Use detailed logging mechanisms to capture and analyze exceptions for debugging purposes.

