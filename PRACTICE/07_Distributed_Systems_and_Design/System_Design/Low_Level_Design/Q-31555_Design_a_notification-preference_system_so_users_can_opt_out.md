---
type: "practice"
difficulty: "Intermediate"
---

# Design a notification-preference system so users can opt out per channel and per category.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pub Sub Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Pub_Sub_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a notification-preference system that allows users to opt-out per channel and per category, we need to ensure flexibility, performance, and ease of management while considering common scenarios like user updates and data consistency. This LLD will focus on the key components: user preferences storage, preference retrieval, and update mechanisms.

#### Triage

The core challenge here is designing a system that can efficiently manage diverse user notifications across multiple channels (e.g., email, push) and categories (e.g., order confirmations, updates). The system should support users opting out of specific combinations of channel and category without affecting other preferences. We need to ensure data consistency and performance, especially when handling large numbers of users and frequent updates.

#### Assumptions
Users can have different preferences for the same category across different channels.
The system must handle both read (retrieve) and write (update) operations efficiently.
Notifications are categorized into predefined groups (e.g., order confirmations, updates).
Channels include email, push notifications, SMS.
Performance requirements: sub-second response times for reading user preferences, under a second for updating.

#### First Principles Solution

Data Model

Each user will have a set of preferences per category and channel combination.

Database Schema

A UserPreferences table with columns like user_id, category_id, channel_id, and is_enabled.

Update Mechanism

Users should be able to update their preferences via an API endpoint.

Retrieval Mechanism

An efficient way to fetch user-specific preferences based on the requested category and channel.

Scalability Considerations

Use a database with good read/write performance, such as PostgreSQL or MongoDB.
Implement caching mechanisms for frequent reads.

Fault Tolerance

Ensure data consistency through transactions during updates.
Have a failover mechanism in place to handle database failures.

#### Implementation
Steps

Database Schema Design

SQL
CREATE TABLE UserPreferences (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES Users(id),
    category_id INTEGER NOT NULL REFERENCES Categories(id),
    channel_id INTEGER NOT NULL REFERENCES Channels(id),
    is_enabled BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

API Endpoints for Preferences

POST /preferences: To update user preferences.
GET /preferences: To retrieve user preferences.

Caching Layer

Use Redis to cache frequently accessed user preferences to reduce database load.

Transaction Management

JAVA
// Example in Java using Spring Data JPA
@Transactional
public void updateUserPreferences(UserPreferences updatedPrefs) {
    UserPreferences existingPrefs = userRepository.findById(updatedPrefs.getId()).orElse(null);
    if (existingPrefs != null) {
        existingPrefs.setIsEnabled(updatedPrefs.getIsEnabled());
        existingPrefs.setUpdated_at(LocalDateTime.now());
        userRepository.save(existingPrefs);
    }
}

Read Performance Optimization

SQL
CREATE INDEX idx_user_category_channel ON UserPreferences(user_id, category_id, channel_id);

#### Closing Summary
and Common Mistakes/Follow-ups
The solution ensures flexibility in managing user preferences while maintaining performance.
Key areas to focus on include database schema design, caching mechanisms, and transaction management.

#### Common mistakes
include overcomplicating the schema or not considering edge cases like stale cache handling.

Do you have any follow-up questions about this design?

**Comparison:**

  | id | user_id | category_id | channel_id | is_enabled |
  |---|----|--------|--------|---------|
  | 1 | 12345 | 101     | 201     | true    |
  | 2 | 12345 | 101     | 202     | false   |
  | 3 | 12345 | 102     | 201     | true    |

**Code:**

```java
// Example of a method in Spring Data JPA to update user preferences
@Transactional
public void updateUserPreferences(UserPreferences updatedPrefs) {
    UserPreferences existingPrefs = userRepository.findById(updatedPrefs.getId()).orElse(null);
    if (existingPrefs != null) {
        existingPrefs.setIsEnabled(updatedPrefs.getIsEnabled());
        existingPrefs.setUpdated_at(LocalDateTime.now());
        userRepository.save(existingPrefs);
    }
}
```

**Example input / output:**

```text
{
  "userId": 12345,
  "categoryIds": [101, 102],
  "channelIds": [201, 202],
  "preferences": [
    {"categoryId": 101, "channelId": 201, "is_enabled": true},
    {"categoryId": 101, "channelId": 202, "is_enabled": false},
    {"categoryId": 102, "channelId": 201, "is_enabled": true}
  ]
}
```

**Data:**

```sql
CREATE TABLE UserPreferences (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES Users(id),
    category_id INTEGER NOT NULL REFERENCES Categories(id),
    channel_id INTEGER NOT NULL REFERENCES Channels(id),
    is_enabled BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_category_channel ON UserPreferences(user_id, category_id, channel_id);
```

**Flow:**

1. Yes
2. No
3. User Makes API Call
4. Is Cache Hit?
5. Cached Data Returned
6. Fetch Data from DB
7. Update Cache
8. Cached Data Updated
9. API Response Created
10. Return Response to User

