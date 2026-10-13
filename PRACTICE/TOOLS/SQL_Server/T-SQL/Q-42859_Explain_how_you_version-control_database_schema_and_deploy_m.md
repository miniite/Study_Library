---
type: "practice"
difficulty: "Deep"
---

# Explain how you version-control database schema and deploy migrations in a CI/CD pipeline for SQL Server.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Temp Tables vs Table Variables](<../../../../TOOLS/SQL_Server/T-SQL/Temp_Tables_vs_Table_Variables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To version-control database schemas and deploy migrations in a CI/CD pipeline for SQL Server, we use tools like Redgate's DbUp or Flyway alongside scripts stored in source control (e.g., Git). This ensures that changes are tracked, reviewed, tested, and deployed consistently across environments. The key steps include creating migration scripts, defining a schema comparison tool, setting up a CI/CD pipeline, and handling database reseeding for data migrations.

#### Triaging the Question

The question is asking how to manage SQL Server database schema changes in a version-controlled manner that integrates with a CI/CD pipeline. This involves several components:

Version Control: Storing database scripts or migration files.
Schema Comparison Tools: Automatically generating diffs between current and desired states.
CI/CD Pipeline Integration: Automating the deployment process.
Data Reseeding: Ensuring data integrity in test/staging environments.

#### Assumptions
We are using SQL Server 2017 or later, which supports T-SQL scripts.
The CI/CD tool is Jenkins for this example.
We have a basic understanding of Git and the branching strategy (e.g., Feature Branch Workflow).
Step-by-Step Solution:
1. Storing Migrations in Source Control

We store our migration files in a dedicated database schema folder within a Git repository. Each file represents a change or set of changes to the database schema, named according to a convention such as 20230401_InitialSetup.sql.

SQL
-- Example: 20230401_InitialSetup.sql
IF NOT EXISTS (SELECT * FROM sys.tables WHERE name = 'Users')
BEGIN
    CREATE TABLE Users (
        Id INT PRIMARY KEY IDENTITY,
        Name NVARCHAR(50) NOT NULL,
        Email NVARCHAR(100) UNIQUE NOT NULL
    );
END;
2. Using a Schema Comparison Tool

Tools like Redgate's Schema Compare or Flyway can be used to compare the current database schema with the desired state and generate migration scripts if needed.

Redgate Schema Compare: Generates a script showing what changes are required between two schemas.
Flyway: Manages versioned migrations, ensuring that only missing versions are applied during deployment.
3. Setting Up CI/CD Pipeline

We set up Jenkins to automatically run SQL migration scripts as part of the build process:

Jenkinsfile:

GROOVY
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    sh 'dotnet restore'
                    sh './build.sh'  # Custom shell script to compile and package app
                }
            }
        }

        stage('Deploy Database Migrations') {
            when { changes 'path/to/database/migrations/**' }  // Watch specific files
            steps {
                sh 'flyway -url=jdbc:sqlserver://localhost:1433;databaseName=MyDatabase -user=myUser -password=myPassword migrate'
            }
        }

        stage('Test') {
            steps {
                script {
                    sh './test.sh'  # Custom test script
                }
            }
        }
    }
}

Flyway Configuration:

PROPERTIES
flyway.url=jdbc:sqlserver://localhost:1433;databaseName=MyDatabase
flyway.user=myUser
flyway.password=myPassword
4. Handling Data Reseeding

For data migrations, we can use scripts or tools like Dapper (a .NET library) to insert test data into the database.

CSHARP
using System.Data.SqlClient;

public class Seeder {
    public static void Seed() {
        using (var conn = new SqlConnection("Data Source=(local);Initial Catalog=MyDatabase;Integrated Security=True"))
        {
            conn.Open();
            var command = new SqlCommand("INSERT INTO Users (Name, Email) VALUES ('John Doe', 'john@example.com');", conn);
            command.ExecuteNonQuery();
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups:

In summary, the key to managing database migrations in a CI/CD pipeline is using tools like Redgate Schema Compare or Flyway for schema management and integration with CI/CD tools like Jenkins. The process ensures that changes are tracked, reviewed, tested, and deployed consistently across environments.

#### Common mistakes
include not having a clear branching strategy, ignoring data reseeding during migrations, and not automating the deployment process. Follow-up questions might involve discussing specific tools in detail or how to handle complex schema changes involving multiple tables and relationships.

**Code:**

```

pipeline { agent any

CODE
Copy
stages {
    stage('Build') {
        steps {
            script {
                sh 'dotnet restore'
                sh './build.sh'
            }
        }
    }

    stage('Deploy Database Migrations') {
        when { changes 'path/to/database/migrations/**' }
        steps {
            sh 'flyway -url=jdbc:sqlserver://localhost:1433;databaseName=MyDatabase -user=myUser -password=myPassword migrate'
        }
    }

    stage('Test') {
        steps {
            script {
                sh './test.sh'
            }
        }
    }
}

}

CODE
Copy

```properties
flyway.url=jdbc:sqlserver://localhost:1433;databaseName=MyDatabase
flyway.user=myUser
flyway.password=myPassword
```

**Flow:**

1. Yes
2. No
3. Source Control
4. Redgate Schema Compare /Flyway
5. Is Database Mismatch?
6. Generate Migration Script
7. Database is up to date
8. Migrate via CI/CD Pipeline
9. Test Environment

