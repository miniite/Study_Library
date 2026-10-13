---
type: "practice"
difficulty: "Intermediate"
---

# Design a like/reaction feature for posts that can handle millions of likes.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Order Tracking Design](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Order_Tracking_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To design a system for handling millions of likes on posts, we need to ensure scalability, reliability, and performance. The key idea is to use a distributed architecture with Redis for real-time updates and a database like PostgreSQL or MySQL for long-term storage. We'll also incorporate caching mechanisms and load balancers.

#### Triaging the Question

The question asks us to design a feature that can handle millions of likes on posts. This requires understanding several aspects:

Scalability: The system must be able to scale horizontally as the number of users increases.
Real-time Updates: Likes should be visible immediately without significant delays.

#### Cons
istency and Durability: We need a reliable way to store like data for future reference.

#### Assumptions
Each post can have multiple likes, each with an associated user ID.
Users can like a post only once (no duplicate likes).
The system should support different types of reactions (like, love, wow).

#### Solution
from First Principles:
1. Database Design:
We need a database to store the posts and their associated likes.
A simple schema might look like this:
SQL
CREATE TABLE Posts (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE Reactions (
    id SERIAL PRIMARY KEY,
    post_id INT REFERENCES Posts(id),
    user_id VARCHAR(255) NOT NULL,
    reaction_type VARCHAR(10) CHECK (reaction_type IN ('like', 'love', 'wow')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(post_id, user_id)
);
The Posts table stores the main content and metadata.
The Reactions table records each reaction, ensuring uniqueness per user-post pair.
2. Real-time Updates:
To provide real-time updates, we can use Redis to store the current like counts for each post.
Whenever a like is added or removed, this count should be updated in Redis immediately.
User Likes
Updates Redis Cache
Post
ReactionTable
RedisCache
3. Load Balancing:
Use a load balancer to distribute requests across multiple instances of the application server, ensuring no single point of failure.
This can be achieved with tools like Nginx or HAProxy.
4. Caching Mechanism:
Cache frequently accessed data using an in-memory cache (like Redis) for faster read operations.
Check Cache
Hit
Miss
UserRequest
CacheStore
UserResponse
Database Fetch
5. Scalability:
Use a horizontally scalable architecture, where the number of instances can be increased as needed.
Load balancers and auto-scaling groups can help manage this.
6. API Design:
API endpoints should handle like additions/removals efficiently:
HTTP
POST /posts/{postId}/reactions
The endpoint will check if the user has already reacted, update Redis and the database accordingly.

#### Closing Summary
+ Common Mistakes/Follow-ups:
Common Mistake: Overlooking the importance of distributed caching or real-time updates. This can lead to stale data or delayed responses.

#### Follow-up
Questions:
How would you handle reactions on a large scale (e.g., billions)?
What about handling reactions in different languages?
How do you ensure consistent data across multiple databases and Redis?
Final Thoughts:

Designing a system that can handle millions of likes requires careful consideration of both the database schema and the application's architecture. By leveraging tools like Redis for real-time updates and PostgreSQL/MySQL for durability, we can build a robust and scalable solution.

**Code:**

```sql
CREATE TABLE Posts (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE Reactions (
    id SERIAL PRIMARY KEY,
    post_id INT REFERENCES Posts(id),
    user_id VARCHAR(255) NOT NULL,
    reaction_type VARCHAR(10) CHECK (reaction_type IN ('like', 'love', 'wow')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(post_id, user_id)
);
```

**Flow:**

1. User Likes
2. Updates Redis Cache
3. Post
4. ReactionTable
5. RedisCache

