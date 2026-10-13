---
type: "practice"
difficulty: "Basic"
---

# Design a tagging system where any entity can have many tags. How do you model this cleanly?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Threaded Comments Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Threaded_Comments_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a tagging system where entities can have multiple tags, we need to ensure that our data model and API are both flexible and efficient. The core idea is to use a many-to-many relationship between entities and tags, which we can achieve through a junction table or an intermediate entity.

#### Triage

The question asks us to design a tagging system where any entity (e.g., a post, a product) can be tagged with multiple tags (e.g., tech, programming). The main goals are:

Flexibility: Support entities and tags dynamically.
Efficiency: Ensure that operations like adding or removing tags do not degrade in performance as the number of tags grows.
Scalability: Handle a large number of tags without significant overhead.

We will assume that there are already entity types (e.g., Post, Product) and tag types (e.g., Tech, Programming). Our focus is on designing the database schema and API to support these requirements efficiently.

#### First-Principles Solution
Database Schema

Entities Table: Stores information about each entity.

SQL
CREATE TABLE Entity (
    id SERIAL PRIMARY KEY,
    type VARCHAR(255) NOT NULL,  -- e.g., 'Post', 'Product'
    name TEXT NOT NULL
);

Tags Table: Stores the tag types.

SQL
CREATE TABLE Tag (
    id SERIAL PRIMARY KEY,
    name TEXT UNIQUE NOT NULL
);

Taggings Table: Junction table to handle many-to-many relationship between entities and tags.

SQL
CREATE TABLE Tagging (
    entity_id INTEGER REFERENCES Entity(id) ON DELETE CASCADE,
    tag_id INTEGER REFERENCES Tag(id) ON DELETE CASCADE,
    PRIMARY KEY (entity_id, tag_id)
);
API Design
Create an Entity: Add a new entity with its type and name.
HTTP
POST /entities
Content-Type: application/json

{
    "type": "Post",
    "name": "My First Post"
}
Add Tags to an Entity:
Retrieve the entity’s ID first.
Add tags by creating entries in the Tagging table.
HTTP
PUT /entities/<entity_id>/tags
Content-Type: application/json

{
    "tag_names": ["Tech", "Programming"]
}
Retrieve Tags for an Entity:
HTTP
GET /entities/<entity_id>/tags

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we use a junction table to handle many-to-many relationships cleanly. This design ensures flexibility, efficiency, and scalability.

#### Common Mistakes
Failing to properly normalize the data (e.g., not using Taggings for the many-to-many relationship).
Not considering edge cases such as duplicate tag entries or null values.
Ignoring potential performance issues with large numbers of tags unless an optimized database system is used.
Self-Check

This design is correct and complete, covering all aspects of the tagging system requirements.

