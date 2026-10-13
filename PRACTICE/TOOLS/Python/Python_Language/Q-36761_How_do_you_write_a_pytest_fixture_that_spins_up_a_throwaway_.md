---
type: "practice"
difficulty: "Deep"
---

# How do you write a pytest fixture that spins up a throwaway Postgres (or container) per test session and rolls back per test, and why is transaction rollback faster than truncate?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Generators and Streaming ETL](<../../../../TOOLS/Python/Python_Language/Generators_and_Streaming_ETL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand how to set up a pytest fixture that creates a temporary Postgres database instance per test session and ensures that each test starts in a clean state by rolling back any changes made during the test execution. Additionally, we will delve into why transaction rollback is generally faster than using TRUNCATE for cleaning up data.

#### Triage

The question is essentially asking how to set up a pytest fixture to manage Postgres databases efficiently per test session and understand the performance differences between transaction rollbacks and TRUNCATE. We need to assume that we are working within a Python environment with access to pytest, Docker, and possibly a service like docker-compose for spinning up containers.

#### First-Principles Solution
Set Up Docker Compose: Use docker-compose to define and run a Postgres container.
Create the Fixture: Write a pytest fixture that:
Starts the Postgres container.
Initializes or connects to the database.
Rolls back any changes made during each test.
Cleans up resources after tests are completed.

#### Implementation

Here’s a step-by-step implementation:

PYTHON
import docker
import os
import psycopg2
import pytest

@pytest.fixture(scope='session')
def postgres_container():
    # Define the Docker Compose configuration for Postgres
    compose_file = """
version: '3'
services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: testuser
      POSTGRES_PASSWORD: testpass
      POSTGRES_DB: testdb
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql/data # Optional: persist data between runs
    networks:
      - network

networks:
  network:
    driver: bridge
    """

    with open('docker-compose.yml', 'w') as f:
        f.write(compose_file)

    client = docker.from_env()
    container = client.containers.run(
        image='postgres',
        environment=['POSTGRES_USER=testuser', 'POSTGRES_PASSWORD=testpass', 'POSTGRES_DB=testdb'],
        ports={'5432/tcp': 5432},
        detach=True,
        network='network'
    )

    yield container

    # Cleanup
    client.containers.stop(container.id)
    client.containers.remove(container.id)

@pytest.fixture(scope='function')
def db(postgres_container):
    conn = psycopg2.connect(
        dbname="testdb",
        user="testuser",
        password="testpass",
        host="localhost",
        port=5432
    )

    def teardown():
        with conn.cursor() as cur:
            # Rollback all transactions to clean up the state of the database
            cur.execute("ROLLBACK")
        conn.close()

    pytest.atexit.register(teardown)
    yield conn

Why Transaction Rollback is Faster than TRUNCATE
Transaction Rollback: This method involves rolling back any changes made within a transaction. It's generally faster because it doesn't require destroying and recreating the table structure or data.
TRUNCATE: This command drops all rows from the table but keeps the table and its indexes intact, making it faster than DELETE. However, it can be less granular in terms of cleanup compared to rolling back a transaction.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes

Not properly closing connections.
Failing to clean up resources after tests are completed.
Misconfiguring Docker Compose or docker-compose.yml.

#### Follow-Ups

Discussing alternatives like using a lightweight in-memory database for unit testing.
Exploring more advanced setup strategies, such as running multiple databases per test suite.

