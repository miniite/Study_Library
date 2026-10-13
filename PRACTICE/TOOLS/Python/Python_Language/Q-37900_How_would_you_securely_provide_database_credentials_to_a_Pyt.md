---
type: "practice"
difficulty: "Basic"
---

# How would you securely provide database credentials to a Python pipeline in production instead of hardcoding them, and where might they come from?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Credential Handling](<../../../../TOOLS/Python/Python_Language/Credential_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely provide database credentials to a Python pipeline in production without hardcoding them, we need to consider best practices like environment variables, encrypted secrets management tools, or secure vaults. The key is to avoid exposing sensitive information directly in the codebase.

#### Triaging the Question

The core of this question revolves around securing sensitive data such as database credentials. Specifically:

Avoid Hardcoding: Directly embedding credentials in the Python source code is a security risk.
Security Sources: Where might these secrets come from?
Secure Storage and Retrieval: How can we safely store and retrieve these credentials?

#### First Principles Solution

Environment Variables: One common method is to use environment variables stored outside your application codebase. This approach ensures that sensitive data remains separate from the source code, reducing the risk of accidental exposure.

Why Environment Variables?
Easy to set up
Can be accessed by any process on the server
Simple to change without redeploying code

Secrets Management Tools: For more robust security, tools like HashiCorp Vault or AWS Secrets Manager can store and manage credentials securely.

Why Secrets Managers?
Provide fine-grained access controls
Offer secure credential rotation mechanisms
Can handle complex secrets with versions and fallbacks

#### Example
Setup Using Environment Variables:

Store the database credentials as environment variables:

BASH
export DB_USER="your_username"
export DB_PASSWORD="your_password"

Access these variables in your Python code:

PYTHON
import os

db_user = os.getenv('DB_USER')
db_password = os.getenv('DB_PASSWORD')

# Connecting to the database using psycopg2 (for PostgreSQL)
from sqlalchemy import create_engine
engine = create_engine(f"postgresql://{db_user}:{db_password}@localhost:5432/your_db")

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Directly hardcoding credentials in source files.

#### Follow-Up
Discuss how to handle different environments (development, staging, production) using environment-specific variables.

For a fresher-level response, we would keep the example simple and focus on the immediate security concern of not hardcoding sensitive information. For a more experienced engineer, you might delve into more complex setups involving secrets managers, key rotation policies, and secure access controls.

**Code:**

```python
import os

# Accessing environment variables in Python
db_user = os.getenv('DB_USER')
db_password = os.getenv('DB_PASSWORD')

# Example of connecting to a PostgreSQL database using SQLAlchemy
from sqlalchemy import create_engine
engine = create_engine(f"postgresql://{db_user}:{db_password}@localhost:5432/your_db")
```

