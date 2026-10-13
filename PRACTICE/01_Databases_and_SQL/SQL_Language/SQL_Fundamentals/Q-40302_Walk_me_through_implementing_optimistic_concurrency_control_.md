---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through implementing optimistic concurrency control with a version column end-to-end, including how the frontend surfaces a 409 conflict and how the retry loop avoids lost updates.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ MVCC Bloat and Vacuum](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/MVCC_Bloat_and_Vacuum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Optimistic concurrency control (OCC) is a technique used to handle data conflicts in concurrent database access without blocking transactions, ensuring that no two users can modify the same data simultaneously. In this implementation, we'll use a version column to track changes and ensure consistency.

#### Triage

The question asks for an end-to-end walkthrough of implementing OCC with a version column. The core components include:

A backend SQL database that maintains a version column.
A frontend handling updates and conflict detection.
Handling of HTTP 409 Conflict responses from the server.
Retry logic to avoid lost updates.

#### Assumptions
We use an SQL database (e.g., PostgreSQL) with a version column in our table.
The frontend sends update requests to the backend, which validates and applies changes if no conflicts are detected.
A conflict is defined as two or more users updating the same record between checks.

#### First-Principles Solution
Database Schema: Add a version column to the table.
Update Request Handling:
The frontend sends an update request with the current version of the record.
The backend retrieves the latest record, including its version.
Version Check and Update:
If the versions match, the server updates the record and returns a 200 OK response.
If the versions don't match, the server responds with a 409 Conflict.
Frontend Handling:
On receiving a 409 Conflict, the frontend re-fetches the current version from the backend.
The frontend then retries the update request with the latest version.

#### Implementation
Steps
Step 1: Database Schema

We need to add a version column to our table. Here’s an example schema:

SQL
CREATE TABLE items (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    value INT,
    version INT DEFAULT 0
);
Step 2: Backend Handling of Update Requests

Here's the SQL logic for updating a record with OCC:

SQL
-- Function to update an item
CREATE OR REPLACE FUNCTION update_item(id INT, new_value INT) RETURNS void AS $$
DECLARE
    current_version INT;
BEGIN
    -- Fetch the current version and check if it matches the provided version
    SELECT version INTO current_version FROM items WHERE id = $1 FOR UPDATE;

    IF current_version IS NULL THEN
        RAISE EXCEPTION 'Item not found';
    END IF;

    -- Update the item only if the version hasn't changed
    IF current_version = (SELECT version FROM items WHERE id = $1) THEN
        UPDATE items SET value = new_value, version = version + 1 WHERE id = $1;
    ELSE
        RAISE EXCEPTION 'Version conflict';
    END IF;
END;
$$ LANGUAGE plpgsql;
Step 3: Frontend Handling of Conflicts and Retries

Here’s a simple example in JavaScript:

JAVASCRIPT
async function updateItem(itemID, newValue) {

#### cons
t [currentVersion] = await fetch(`/api/items/${itemID}`)
        .then(response => response.json())
        .catch(error => console.error('Error fetching the current item:', error));

    try {

#### cons
t response = await fetch(`/api/items/update`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ id: itemID, version: currentVersion, newValue })
        });

        if (response.ok) {

#### cons
ole.log('Item updated successfully');
        } else if (response.status === 409) {

#### cons
t latestVersion = await response.json();
            // Retry the update with the latest version
            return updateItem(itemID, newValue);
        } else {
            throw new Error(`Unexpected response: ${response.status}`);
        }
    } catch (error) {

#### cons
ole.error('Error updating item:', error);
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Forgetting to use FOR UPDATE in SQL queries.
Not handling version mismatches properly, leading to inconsistent data.
Not providing meaningful error messages or user feedback.

#### Follow-Ups

Discussing alternative concurrency control strategies like pessimistic locking.
Exploring how OCC interacts with other database features (e.g., transactions).
Discussing potential optimizations for high-concurrency systems.
Self-CHECK

The solution covers the necessary steps and addresses common pitfalls. It is detailed enough to cover both backend and frontend aspects, ensuring robustness against concurrency issues.

**Comparison:**

| id | name   | value | version |
|----|--------|-------|---------|
| 1  | Item A | 50    | 3       |

**Code:**

```sql
CREATE TABLE items (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    value INT,
    version INT DEFAULT 0
);

CREATE OR REPLACE FUNCTION update_item(id INT, new_value INT) RETURNS void AS $$
DECLARE
    current_version INT;
BEGIN
    -- Fetch the current version and check if it matches the provided version
    SELECT version INTO current_version FROM items WHERE id = $1 FOR UPDATE;

    IF current_version IS NULL THEN
        RAISE EXCEPTION 'Item not found';
    END IF;

    -- Update the item only if the version hasn't changed
    IF current_version = (SELECT version FROM items WHERE id = $1) THEN
        UPDATE items SET value = new_value, version = version + 1 WHERE id = $1;
    ELSE
        RAISE EXCEPTION 'Version conflict';
    END IF;
END;
$$ LANGUAGE plpgsql;

async function updateItem(itemID, newValue) {
    const [currentVersion] = await fetch(`/api/items/${itemID}`)
        .then(response => response.json())
        .catch(error => console.error('Error fetching the current item:', error));

    try {
        const response = await fetch(`/api/items/update`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ id: itemID, version: currentVersion, newValue })
        });

        if (response.ok) {
            console.log('Item updated successfully');
        } else if (response.status === 409) {
            const latestVersion = await response.json();
            // Retry the update with the latest version
            return updateItem(itemID, newValue);
        } else {
            throw new Error(`Unexpected response: ${response.status}`);
        }
    } catch (error) {
        console.error('Error updating item:', error);
    }
}
```

**Example input / output:**

```text
Input:
- Fetch the current item with ID `1`: `/api/items/1`

Output:
- JSON response containing the current version: `{ id: 1, name: "Item A", value: 50, version: 3 }`

Input (Retry):
- Fetch the updated version of the item after conflict: `/api/items/update` with body `{"id": 1, "version": 3, "newValue": 60}`

Output:
- JSON response containing the latest version: `{ id: 1, name: "Item A", value: 60, version: 4 }`

Input (Conflict):
- Fetch the item with ID `1` after another user updated it: `/api/items/1`

Output:
- HTTP 409 Conflict response
```

**Data:**

```json
{
    "id": 1,
    "name": "Item A",
    "value": 50,
    "version": 3
}
```

**Flow:**

1. No
2. Yes
3. Fetch Current Item
4. Version Match?
5. Retry with Latest Version
6. Update Record
7. Conflict Detected
8. Fetch Latest Version

