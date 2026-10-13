---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd standardize and govern the use of application-level row filtering versus database-enforced RLS across teams, so security isn't silently dependent on app correctness.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Database Access Control and Auditing](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/Database_Access_Control_and_Auditing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To standardize and govern the use of application-level row filtering (ALRF) versus database-enforced Row-Level Security (RLS), we need to ensure that data access controls are consistently applied, reducing dependency on application correctness and enhancing overall security. This involves a multi-faceted approach, including policy standardization, governance, and enforcement mechanisms.

Firstly, the core concept is to centralize control where it makes sense: RLS should be used when possible because it can enforce security policies at the database level, making them more robust and harder to bypass. ALRF remains useful for certain scenarios but must be strictly governed.

The key steps involve:

Defining a clear policy that distinguishes between when ALRF and RLS are appropriate.
Implementing mechanisms to audit, monitor, and enforce these policies consistently across all applications.
Providing tools or frameworks that make it easy for developers to implement both methods correctly.

**Comparison:**

```
Policy	Applicability	Description
RLS	Highly sensitive data	Enforces security policies at the database level, making them harder to bypass.
ALRF	Less sensitive or less critical scenarios	Leverages application logic for row-level filtering but requires careful governance and monitoring.
```

**Code:**

```python
# Example Python code snippet for applying RLS in a database (using SQLAlchemy ORM)
from sqlalchemy import Column, Integer, String, Table, ForeignKeyConstraint
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String)

class Data(Base):
    __tablename__ = 'data'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer)
    content = Column(String)

    # Define a foreign key constraint to enforce RLS
    __table_args__ = (ForeignKeyConstraint([user_id], [User.id]),)

# Example of applying RLS in the database schema
def apply_rls(engine):
    with engine.connect() as conn:
        conn.execute("ALTER TABLE data ENABLE ROW LEVEL SECURITY")
        conn.execute("""
            CREATE POLICY my_policy ON data
            FOR SELECT TO users
            USING (user_id = current_setting('user.id')::int);
        """)
```

**Example input / output:**

```text
# Example usage of the RLS policy in a Python application
from sqlalchemy.orm import sessionmaker, Session

Session = sessionmaker(bind=engine)
session = Session()

user_id = 123  # User ID from authentication context
data_rows = session.query(Data).filter_by(user_id=user_id).all()
```

**Data:**

```text
- `users`: Contains user information.
- `data`: Stores data that may be sensitive and needs row-level security.
```

**Flow:**

1. Yes
2. No
3. Define RLS Policy
4. Is Data Highly Sensitive?
5. Enable Database-Level RLS
6. Leverage ALRF withGovernance
7. Implement and Audit Policies
8. Develop Tools for EasyImplementation
9. Monitor and Enforce PoliciesConsistently

