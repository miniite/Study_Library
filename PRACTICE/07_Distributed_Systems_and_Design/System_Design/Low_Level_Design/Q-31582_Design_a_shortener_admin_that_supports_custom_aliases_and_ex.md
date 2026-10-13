---
type: "practice"
difficulty: "Intermediate"
---

# Design a shortener admin that supports custom aliases and expiry. How does this change your earlier data model?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tenant Isolation in Repositories](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an admin system for managing URL shorteners with support for custom aliases and expiring URLs, we need to rethink our data model to accommodate these features while ensuring efficient operations. The core idea is to balance between storage efficiency, performance, and the flexibility required by the admin interface.

#### Triaging the Question

The interviewer is probing into your ability to refactor a data model in response to new requirements. Specifically:

Custom Aliases: Users should be able to define their own custom aliases for shortened URLs.
Expiry Support: Some shortened URLs should expire after a certain period, adding time-sensitive logic.

#### Assumptions
and Initial Setup

Assume the following:

The existing URL shortening system uses a simple hash map or database table where each key is a unique identifier (e.g., "abcd1234") and the value is the full URL.
New requirements: Custom aliases are allowed, and some URLs need to expire.
Step-by-Step Solution

We start by defining our data model components:

Data Model Components
URL Records Table: Stores the shortened keys along with their corresponding long URLs.
Alias Mapping Table: Maps custom aliases to specific URL records.
Expiry Timestamps Table: Tracks when each URL record should expire.
Step 1: Define the Tables

We can define three tables as follows:

SQL
CREATE TABLE url_records (
    id SERIAL PRIMARY KEY,
    short_key VARCHAR(20) UNIQUE NOT NULL,  -- Unique identifier for the shortened URL
    long_url TEXT NOT NULL,                -- The full URL to be shortened
    expiry_timestamp TIMESTAMP            -- Timestamp when the URL should expire (NULL if no expiry)
);

CREATE TABLE alias_mapping (
    id SERIAL PRIMARY KEY,
    alias VARCHAR(10) UNIQUE NOT NULL,     -- Custom alias for the shortened URL
    url_record_id INTEGER REFERENCES url_records(id) ON DELETE CASCADE
);
Step 2: Implementing Custom Aliases and Expiry

To support custom aliases and expiry:

Insertion: When a user creates a new URL or assigns an alias, we insert records into both url_records and possibly alias_mapping.
Expiry Handling: We schedule deletions for expiring URLs based on the expiry_timestamp.
Step 3: Querying Shortened URLs

To retrieve the full URL using either the short key or a custom alias:

SQL
SELECT u.long_url FROM url_records u
LEFT JOIN alias_mapping a ON u.id = a.url_record_id
WHERE (u.short_key = :short_key OR a.alias = :alias) AND (expiry_timestamp IS NULL OR expiry_timestamp > NOW());

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we have:

Introduced two new tables to support custom aliases and expiry.
Ensured efficient queries by using LEFT JOIN with the appropriate conditions.

#### Common Mistakes

Overcomplication: Adding unnecessary complexity to handle every edge case without a clear requirement.
Ignoring Performance Considerations: Not considering how frequent expirations will impact performance.

#### Follow-ups
might include:

How would you optimize this system for high traffic?
What are the implications of storing expiry timestamps on each record versus using a separate table?

**Code:**

```sql
CREATE TABLE url_records (
    id SERIAL PRIMARY KEY,
    short_key VARCHAR(20) UNIQUE NOT NULL,  -- Unique identifier for the shortened URL
    long_url TEXT NOT NULL,                -- The full URL to be shortened
    expiry_timestamp TIMESTAMP            -- Timestamp when the URL should expire (NULL if no expiry)
);

CREATE TABLE alias_mapping (
    id SERIAL PRIMARY KEY,
    alias VARCHAR(10) UNIQUE NOT NULL,     -- Custom alias for the shortened URL
    url_record_id INTEGER REFERENCES url_records(id) ON DELETE CASCADE
);
```

**Example input / output:**

```text
-- Inserting a new URL with an expiry date
INSERT INTO url_records (short_key, long_url, expiry_timestamp)
VALUES ('abcd1234', 'http://example.com', NOW() + INTERVAL '7 days');

-- Mapping a custom alias to the URL record
INSERT INTO alias_mapping (alias, url_record_id)
VALUES ('myurl', 1);

-- Retrieving the full URL using short key or alias
SELECT u.long_url FROM url_records u
LEFT JOIN alias_mapping a ON u.id = a.url_record_id
WHERE (u.short_key = 'abcd1234' OR a.alias = 'myurl') AND (expiry_timestamp IS NULL OR expiry_timestamp > NOW());
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Expiry > Now()
6. Expiry <= Now()
7. Expiry > Now()
8. Expiry <= Now()
9. User Request
10. Is Short Key Provided?
11. Retrieve URL using short key
12. Is Alias Provided?
13. Retrieve URL using alias
14. Return Error: No valid inputprovided
15. Check Expiry Timestamp
16. Return Long URL
17. Delete Record and Return Error
18. Check Expiry Timestamp
19. Return Long URL
20. Delete Record and Return Error

