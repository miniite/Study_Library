---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd represent a workflow/state-machine entity in the schema so invalid state transitions are impossible at the DB layer.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Modeling Relationships and Schemas](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Modeling_Relationships_and_Schemas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a database schema that ensures valid state transitions, we need to model the workflow as a state machine directly within our database schema. The core idea is to use an enum data type for the state field and enforce constraints on the state transitions via triggers or check constraints. Let's break this down step-by-step.

Firstly, what are we trying to achieve? We want to ensure that only valid state transitions can occur in the workflow. For instance, if a task can only transition from "Pending" to "In Progress", then it should not be possible for it to go directly from "Pending" to "Completed". By enforcing these rules at the database level, we minimize the risk of application bugs leading to invalid states.

Now, let's consider our assumptions:

The workflow has a defined set of states.
We need to track transitions between states accurately and reliably.
Valid state transitions are predefined and known ahead of time.

To implement this in practice, we'll follow these steps:

Define the States as an Enum: In most modern SQL databases like PostgreSQL, MySQL, or SQL Server, enums can be used to represent valid states. We define a column with an enum type that lists all possible states for our workflow.
Track State Transitions Explicitly (Optional): If we want to keep track of historical state changes, we can create a separate table for logging transitions.
Enforce Valid Transitions via Triggers: By creating triggers or check constraints, we can prevent invalid state transitions from occurring.

Here’s how we might model this in SQL:

SQL
-- Define the enum type for states
CREATE TYPE workflow_state AS ENUM ('Pending', 'In Progress', 'Completed');

-- Create a table with an enum column for states
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    state workflow_state NOT NULL DEFAULT 'Pending',
    transition_log_id INTEGER REFERENCES task_transitions(id)
);

-- Optional: Table to log transitions (for history tracking)
CREATE TABLE task_transitions (
    id SERIAL PRIMARY KEY,
    task_id INTEGER NOT NULL,
    from_state workflow_state NOT NULL,
    to_state workflow_state NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Next, we'll implement a trigger or check constraint to ensure that only valid transitions occur:

SQL
-- Example of using a trigger to enforce state transition rules
CREATE OR REPLACE FUNCTION enforce_valid_transition()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.state = 'Completed' AND OLD.state != 'In Progress' THEN
        -- Raise an error if the transition is invalid
        RAISE EXCEPTION 'Invalid transition: % -> %', OLD.state, NEW.state;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_valid_transition BEFORE UPDATE ON tasks
FOR EACH ROW EXECUTE FUNCTION enforce_valid_transition();

In this example, the trigger enforce_valid_transition ensures that a task can only be set to 'Completed' if its previous state was 'In Progress'. If an invalid transition is attempted, a constraint violation error will occur.

This approach not only ensures data integrity but also makes our schema more robust and easier to maintain. Common mistakes include forgetting to define the enum properly, not tracking historical transitions where required, or incorrectly writing the trigger logic.

**Code:**

```sql
CREATE TYPE workflow_state AS ENUM ('Pending', 'In Progress', 'Completed');

CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    state workflow_state NOT NULL DEFAULT 'Pending',
    transition_log_id INTEGER REFERENCES task_transitions(id)
);

CREATE TABLE task_transitions (
    id SERIAL PRIMARY KEY,
    task_id INTEGER NOT NULL,
    from_state workflow_state NOT NULL,
    to_state workflow_state NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION enforce_valid_transition()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.state = 'Completed' AND OLD.state != 'In Progress' THEN
        RAISE EXCEPTION 'Invalid transition: % -> %', OLD.state, NEW.state;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_valid_transition BEFORE UPDATE ON tasks
FOR EACH ROW EXECUTE FUNCTION enforce_valid_transition();
```

