---
type: "practice"
difficulty: "Basic"
---

# How do you create a new database and a new user, then grant that user access in PostgreSQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Row-level Security](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Row-level_Security.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up a new database and a new user with appropriate access in PostgreSQL, we need to follow a few steps from first principles. This involves creating the database, adding a new user, and granting the necessary privileges to that user. Let's break this down step-by-step.

First, we will start by setting up a basic understanding of what each step entails:

Creating a new database: We define the name, properties (like encoding and locale), and optionally add some initial settings.
Creating a new user: We need to specify a username and password. Optionally, we can also set other attributes like role type or superuser status.

Now let's dive into each step in detail.

Creating a New Database
Creating a New User
Granting Access

#### Triaging the Question

The core of this question is about understanding and executing PostgreSQL commands to manage databases and users effectively. The interviewer might be probing your familiarity with basic PostgreSQL administration commands, including CREATE DATABASE, CREATE USER, and GRANT statements.

#### Assumptions

You are using a Unix-like environment (like Linux or macOS) where you can execute SQL commands.
You have administrative privileges to create databases and users.
Step-by-Step Solution
1. Creating a New Database

To create a new database, we use the CREATE DATABASE command. Let's say we want to create a database called mydatabase.

SQL
-- Connect to your PostgreSQL server (e.g., using psql)
psql -U postgres

-- Create the database
CREATE DATABASE mydatabase;
2. Creating a New User

Next, we need to create a new user who will have access to this database. For example, let's create a user named myuser.

SQL
-- Grant superuser privileges (optional but useful for initial setup)
ALTER USER myuser WITH SUPERUSER;

-- Create the user with a password
CREATE USER myuser WITH PASSWORD 'mypassword';
3. Granting Access

Now that we have our database and user, we need to grant them access to the mydatabase. We can do this by using the GRANT statement.

SQL
-- Connect to the database you just created (mydatabase)
\c mydatabase

-- Grant all privileges on the database to the new user
GRANT ALL PRIVILEGES ON DATABASE mydatabase TO myuser;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we first created a database named mydatabase, then added a superuser with the name myuser who has full privileges over this database. It's important to note that granting superuser status is optional but can be useful for initial setup or troubleshooting.

#### Common mistakes
include:

Forgetting to specify a password when creating a user.
Not granting sufficient privileges, which could limit the user’s capabilities.
Connecting directly to postgres instead of mydatabase.
Additional Considerations
Security: Always use strong passwords and consider role separation for better security practices.
Permissions: Depending on the application requirements, you might need to grant specific permissions rather than ALL PRIVILEGES.
Environment Setup: Ensure that your PostgreSQL server is correctly configured and accessible.

**Code:**

```sql
-- Example SQL commands
CREATE DATABASE mydatabase;
CREATE USER myuser WITH PASSWORD 'mypassword';
GRANT ALL PRIVILEGES ON DATABASE mydatabase TO myuser;
```

