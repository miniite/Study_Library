---
type: "practice"
difficulty: "Basic"
---

# How would you design a SQL schema linking users, their conversations, and the individual messages in each conversation?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validating Join Cardinality](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Validating_Join_Cardinality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To design an SQL schema that links users, their conversations, and the individual messages within those conversations, we need to create three primary tables: Users, Conversations, and Messages. Each table will have its own structure with relevant columns. The key to linking these tables is through foreign keys, which ensure referential integrity.

#### Triage
This question is asking you to design a relational database schema using SQL to manage user interactions in conversations. The interviewer wants to see your understanding of normalization, relationships between entities, and the use of foreign keys to maintain data integrity.

Step-by-Step Solution:

Define Entities: Users participate in conversations which contain messages.
Create Tables: Define columns for each entity (table).
Establish Relationships: Use foreign keys to link tables appropriately.
Step 1: Define the Users Table

This table will store information about users.

SQL
CREATE TABLE Users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Step 2: Define the Conversations Table

This table will store details about each conversation.

SQL
CREATE TABLE Conversations (
    conversation_id INT PRIMARY KEY AUTO_INCREMENT,
    topic VARCHAR(255) NOT NULL,
    started_by_user_id INT,
    start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (started_by_user_id) REFERENCES Users(user_id)
);
Step 3: Define the Messages Table

This table will store individual messages within conversations.

SQL
CREATE TABLE Messages (
    message_id INT PRIMARY KEY AUTO_INCREMENT,
    sender_user_id INT NOT NULL,
    conversation_id INT NOT NULL,
    content TEXT NOT NULL,
    sent_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (sender_user_id) REFERENCES Users(user_id),
    FOREIGN KEY (conversation_id) REFERENCES Conversations(conversation_id)
);

#### Closing Summary

The schema consists of three tables: Users for user information, Conversations to track the conversations initiated by users, and Messages for storing individual messages. Foreign keys ensure that each message is associated with a valid conversation and user.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to include primary keys or foreign keys.
Mistake 2: Not normalizing the schema, leading to redundancy (e.g., storing user information in both Users and Messages).

#### Follow-Up
Discuss potential optimizations like indexing for performance, or consider adding more columns or constraints as needed.

**Code:**

```sql
CREATE TABLE Users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE Conversations (
    conversation_id INT PRIMARY KEY AUTO_INCREMENT,
    topic VARCHAR(255) NOT NULL,
    started_by_user_id INT,
    start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (started_by_user_id) REFERENCES Users(user_id)
);

CREATE TABLE Messages (
    message_id INT PRIMARY KEY AUTO_INCREMENT,
    sender_user_id INT NOT NULL,
    conversation_id INT NOT NULL,
    content TEXT NOT NULL,
    sent_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (sender_user_id) REFERENCES Users(user_id),
    FOREIGN KEY (conversation_id) REFERENCES Conversations(conversation_id)
);
```

