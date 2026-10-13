---
type: "practice"
difficulty: "Intermediate"
---

# How would you use pytest fixtures to spin up a clean database per test for FastAPI integration tests?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database Access and Tool Calls](<../../../../TOOLS/Python/FastAPI/Database_Access_and_Tool_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure your FastAPI application’s integration tests are isolated and that each test starts with a clean state, using pytest fixtures is an excellent practice. In this context, we'll create a fixture to set up a clean database environment before every test runs. This approach helps avoid side effects between tests and ensures robustness.

Firstly, let's outline what the question really asks: We need to understand how to leverage pytest fixtures to manage database state across different test cases in a FastAPI application. The key idea is that each test should have an isolated environment where it can operate without interfering with others.

To tackle this, we'll assume you already have a basic setup of FastAPI and a PostgreSQL database for your application. We will use the pytest-postgresql plugin to manage the database instance within our tests. Here are the steps:

Install necessary plugins: Ensure you have pytest, pytest-postgresql, and any other relevant dependencies installed.
Set up the fixture: Create a fixture function that initializes the database for each test, drops all tables at setup, and fills it with initial data if needed.

Let's dive into the implementation details:

Step-by-Step Implementation

Install Dependencies:

BASH
pip install pytest pytest-postgresql psycopg2-binary

Define the Fixture in a Test File: Create a test file, for example, test_integration.py, and define the fixture there.

Create the Database Setup and Teardown Logic: Here’s how you can implement it:

PYTHON
import pytest
from sqlalchemy import create_engine
from fastapi.testclient import TestClient

# Assuming your database URL is stored in an environment variable for security reasons.
DATABASE_URL = "postgresql://user:password@localhost/test_db"

@pytest.fixture(scope="session")
def test_db():
    """Cleanup and setup the database before all tests."""
    engine = create_engine(DATABASE_URL)
    with engine.connect() as connection:
        # Start a transaction
        trans = connection.begin()

        try:
            # Drop all tables to ensure a clean state
            from your_module import Base  # Import your models or base classes

            Base.metadata.drop_all(connection)

            # Recreate the schema
            Base.metadata.create_all(connection)

            yield connection

        except Exception as e:
            print(f"An error occurred: {e}")
            trans.rollback()
        finally:
            trans.close()

@pytest.fixture(scope="function")
def client(test_db):
    """Provide a test client for each function scoped test."""
    from your_app import app  # Import the FastAPI application

    with TestClient(app) as client:
        yield client
Write Tests: Now you can write tests that use these fixtures to ensure they operate on clean data.

#### Example
Test
PYTHON
def test_create_user(client):
    response = client.post("/users", json={"username": "testuser"})
    assert response.status_code == 201

    user_id = response.json()["id"]

    # Additional assertions can be added here to check the database state.

#### Closing Summary

By using pytest fixtures, we ensure that each test starts with a clean database state, which is crucial for isolation and robust testing. This approach helps in avoiding issues like side effects between tests and ensures that your application behaves as expected under different conditions.

#### Common Mistakes/Follow-Ups
Incorrect Scope: Not understanding the difference between function and session scopes can lead to unexpected behavior.
Lack of Error Handling: Failing to handle exceptions properly in fixtures might cause tests to fail unexpectedly.
Dependency on Environment Variables: Ensuring that your database URL is correctly set as an environment variable or hard-coded securely.

