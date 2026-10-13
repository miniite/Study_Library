---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a table and query to answer 'which feature flags is this user in?' where flags target by percentage rollout using a deterministic hash of user_id. How do you keep the evaluation stable as the percentage changes?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Aggregation Query Practice](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of determining which feature flags a user is in, given that these flags are targeted by percentage rollout using a deterministic hash of user_id, we need to design an efficient and scalable solution. The key challenge is ensuring that as the percentage rollouts change, the evaluation remains stable and consistent.

#### Triage

The question asks us to:

Design a table schema.
Write a query to fetch the relevant feature flags for a given user.
Ensure the stability of evaluations even when the percentage rollouts are adjusted.

We will make the following assumptions:

We have a FeatureFlags table that stores the metadata of each flag, including its unique identifier and rollout percentage.
The UserRollouts table contains the deterministic hash of user_id and the relevant feature flags it is assigned to.
Step-by-Step Solution
1. Table Design

To efficiently manage user rollouts based on a percentage, we need two primary tables:

FeatureFlags: Stores metadata about each flag.
UserRollouts: Maps users to their assigned feature flags.

Table Schema:

SQL
CREATE TABLE FeatureFlags (
    id INT PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE,
    rollout_percentage DECIMAL(5, 4) NOT NULL CHECK (rollout_percentage BETWEEN 0 AND 1),
    hash_key VARCHAR(64)
);

CREATE TABLE UserRollouts (
    user_id BIGINT,
    flag_id INT,
    PRIMARY KEY (user_id, flag_id),
    FOREIGN KEY (flag_id) REFERENCES FeatureFlags(id)
);

Explanation:

FeatureFlags: Stores the identifier and rollout percentage of each feature.
UserRollouts: Maps users to their assigned feature flags using a deterministic hash of user_id.
2. Query Design

To determine which feature flags a given user is in, we need to compare the deterministic hash of the user's ID with the rollouts.

Query:

SQL
SELECT
    ff.id,
    ff.name
FROM FeatureFlags ff
JOIN UserRollouts ur ON ur.flag_id = ff.id
WHERE ur.user_id = [user_id] AND (RAND() * 100) < ff.rollout_percentage;

Explanation:

We join the FeatureFlags and UserRollouts tables on flag_id.
The query filters rows based on whether a random number generated between 0 and 100 is less than the feature's rollout percentage.
This ensures that users are matched to their respective features probabilistically according to the given percentage.
3. Stability of Evaluations

To ensure stability as the percentage rollouts change, we need to update the UserRollouts table whenever the percentages are adjusted.

Update Strategy:

Calculate New Hashes: For each user, recalculate the deterministic hash using a consistent hashing function based on the new rollout percentages.
Assign Feature Flags: Assign feature flags to users based on their new hashes and the updated rollouts in UserRollouts.
Mark Users for Update: Maintain a list of users who need to be reassigned due to changes in percentages.

#### Example
Dry-Run:

Given:

User with user_id = 12345 has a deterministic hash.
Feature flag with id = 1 and rollout_percentage = 0.7.

New rollout percentage for the feature is changed to 0.6.

Steps:

Recalculate the hash of user 12345.
Determine if the user should still be assigned to the feature.
Update UserRollouts accordingly.

#### Closing Summary

By designing a FeatureFlags table and a UserRollouts table, we ensure that users are correctly matched to their feature flags based on randomized rollouts. The stability of evaluations is maintained by reassigning users when rollout percentages change, ensuring consistent behavior without disrupting ongoing user interactions.

#### Common Mistakes/Follow-Ups

Not considering the deterministic nature of user_id hashes.
Failing to account for updates in rollouts without a proper reevaluation mechanism.
Misunderstanding how random number generation affects the evaluation process.

**Comparison:**

```
FeatureFlags
PrimaryKey
id
name
rollout_percentage
hash_key
UserRollouts
FK_flag_id
flag_id
FeatureFlags.id
UserRollouts.user_id
```

**Code:**

```sql
CREATE TABLE FeatureFlags (
    id INT PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE,
    rollout_percentage DECIMAL(5, 4) NOT NULL CHECK (rollout_percentage BETWEEN 0 AND 1),
    hash_key VARCHAR(64)
);

CREATE TABLE UserRollouts (
    user_id BIGINT,
    flag_id INT,
    PRIMARY KEY (user_id, flag_id),
    FOREIGN KEY (flag_id) REFERENCES FeatureFlags(id)
);
```

**Example input / output:**

```text
-- Query to fetch user's feature flags based on their hash and rollouts
SELECT
    ff.id,
    ff.name
FROM FeatureFlags ff
JOIN UserRollouts ur ON ur.flag_id = ff.id
WHERE ur.user_id = 12345 AND (RAND() * 100) < ff.rollout_percentage;
```

**Data:**

```sql
-- Example Data for FeatureFlags Table
INSERT INTO FeatureFlags (id, name, rollout_percentage, hash_key) VALUES
(1, 'FeatureA', 0.7, 'hash_value_1'),
(2, 'FeatureB', 0.3, 'hash_value_2');

-- Example Data for UserRollouts Table
INSERT INTO UserRollouts (user_id, flag_id) VALUES
(12345, 1);
```

**Flow:**

1. Step 1
2. Calculate New Hashes
3. Determine ReassignmentNeeded
4. Update UserRollouts Table
5. Mark Users for Update
6. Re-evaluate Feature Flags

