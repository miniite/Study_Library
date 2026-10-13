---
type: "practice"
difficulty: "Deep"
---

# How would you design indexing and query patterns for a 'seen/unseen notifications' feature where each user reads their own unread set constantly and marks-as-read in bulk?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Indexing Read State at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Indexing_Read_State_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient indexing and query pattern for a "seen/unseen notifications" feature, we need to focus on optimizing read performance while minimizing write operations, especially for marking as read. The key here is to maintain both unread and seen notifications in separate indexes or tables, ensuring that each user can efficiently query their unread notifications without affecting the overall system's scalability.

#### Overview
We will use a combination of database indexing and efficient query patterns to optimize the feature.

#### Triage
The question asks for designing an indexing strategy and query patterns for managing seen/unseen notifications. It probes our understanding of indexing techniques, query optimization, and handling bulk operations efficiently.

#### First-Principles Solution

Database Schema: We will have two primary tables: UserNotifications to store all notifications and UserReadStatus to track read status for each user-notification pair.
Indexing Strategy: Use indexes on the UserNotifications table to quickly identify unread notifications. For the UserReadStatus, use a composite index to efficiently mark as read in bulk.
Query Patterns: Efficiently query unread notifications by leveraging indexes, and batch-mark-as-read operations to minimize write amplification.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The solution involves creating an index on the UserNotifications.read_status column to quickly fetch unread notifications for a user.
Use a composite index (user_id, read_status) in the UserReadStatus table to batch-mark-as-read operations efficiently.

#### Common mistakes
include not considering the write amplification during bulk updates and using too many indexes, which can slow down write operations.

**Comparison:**

| id | user_id | notification_text          | read_status |
|----|---------|----------------------------|-------------|
| 1  | 101     | You have a new message      | false       |
| 2  | 101     | Another unread notification | false       |
| user_id | notification_id | last_read                   |
|---------|-----------------|-----------------------------|
| 101     | 1               | 2023-10-05T14:48:00Z        |
| 101     | 2               | 2023-10-05T14:48:00Z        |

**Code:**

```sql
-- UserNotifications Table Schema
CREATE TABLE UserNotifications (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    notification_text TEXT NOT NULL,
    read_status BOOLEAN DEFAULT FALSE
);

-- UserReadStatus Table Schema
CREATE TABLE UserReadStatus (
    user_id INT NOT NULL,
    notification_id INT NOT NULL,
    last_read TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id, notification_id) REFERENCES UserNotifications(user_id, id),
    PRIMARY KEY (user_id, notification_id)
);
```

**Example input / output:**

```text
Input:
- User ID: 101
- Notifications to mark as read: [1, 2]

Output:
- Updated UserNotifications table:
  - { "id": 1, "user_id": 101, "notification_text": "You have a new message", "read_status": true }
  - { "id": 2, "user_id": 101, "notification_text": "Another unread notification", "read_status": true }

- Updated UserReadStatus table:
  - { "user_id": 101, "notification_id": 1, "last_read": "2023-10-05T14:48:00Z" }
  - { "user_id": 101, "notification_id": 2, "last_read": "2023-10-05T14:48:00Z" }
```

**Data:**

```json
{
  "UserNotifications": [
    { "id": 1, "user_id": 101, "notification_text": "You have a new message", "read_status": false },
    { "id": 2, "user_id": 101, "notification_text": "Another unread notification", "read_status": false }
  ],
  "UserReadStatus": [
    { "user_id": 101, "notification_id": 1, "last_read": "2023-10-05T14:48:00Z" },
    { "user_id": 101, "notification_id": 2, "last_read": "2023-10-05T14:48:00Z" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Marks as Read
4. Mark As Read?
5. Marks read in UserReadStatus
6. Fetch Unread Notificationsfrom UserNotifications
7. UpdateUserNotifications.read_statusto true
8. Notify User of Updates

