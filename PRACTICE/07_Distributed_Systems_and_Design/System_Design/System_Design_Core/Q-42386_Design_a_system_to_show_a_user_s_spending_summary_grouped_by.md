---
type: "practice"
difficulty: "Intermediate"
---

# Design a system to show a user's spending summary grouped by category.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Order Tracking Design](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Order_Tracking_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that shows a user’s spending summary grouped by category, we need to break down the problem into several components and consider various aspects such as data storage, API design, caching, and scalability. The core idea is to efficiently retrieve and aggregate spending data for each category per user.

#### Triage

The question is asking us to design an efficient system that can handle frequent requests for a user's spending summary categorized by their expenses. The key elements we need to consider are:

Data Storage: How to store the spending data.
API Design: What API endpoints will be needed and how they should function.
Caching: To improve response times for frequently accessed summaries.
Scalability: Ensuring the system can handle a growing number of users and their expenses.

#### First-Principles Solution

Data Model

Users have multiple categories (e.g., food, entertainment, transportation).
Each category has associated spending data (amount, timestamp).

API Design

GET /users/:id/summary: Returns the summary of a user’s spending categorized by each category.
PLAINTEXT
{
  "userId": 1,
  "categories": [
    { "categoryName": "food", "totalSpending": 500 },
    { "categoryName": "entertainment", "totalSpending": 300 },
    ...
  ]
}

Database Design

Use a relational database (e.g., PostgreSQL) to store spending data.
SQL
CREATE TABLE Spending (
  id SERIAL PRIMARY KEY,
  userId INT REFERENCES Users(id),
  categoryName VARCHAR(50),
  amount DECIMAL(10, 2),
  timestamp TIMESTAMP
);

Caching Strategy

Use an in-memory cache (e.g., Redis) to store the aggregated spending summaries.
Cache invalidation strategy: Invalidate summary on new or updated spendings.

API Implementation

Fetch data from both the database and cache, combining them as necessary.
Ensure the API can handle concurrent requests efficiently.

Scalability Considerations

Load balancing to distribute incoming requests across multiple servers.
Horizontal scaling by adding more database nodes if needed.

#### Closing Summary

In summary, designing a system for showing a user's spending summary involves creating an efficient data model and API design, implementing caching strategies, and ensuring the system can scale with growing usage. The key is to balance between query performance, memory usage, and overall efficiency.

#### Common Mistakes/Follow-Ups

Mistake: Not considering the impact of frequent updates on cache invalidation.

#### Follow-Up
Discuss a more robust cache invalidation strategy that minimizes stale data issues.

Mistake: Overcomplicating the database schema without proper normalization.

#### Follow-Up
Highlight the importance of using normalized schemas to reduce redundancy and improve performance.

**Comparison:**

```
Spending Table	Description
id (Serial)	Unique identifier for each spending record.
userId (INT, Foreign Key)	The user who made the spending.
categoryName (VARCHAR(50))	Category of the spending (e.g., food, entertainment).
amount (DECIMAL(10, 2))	Amount spent for this category.
timestamp (TIMESTAMP)	Time when the spend occurred.
```

**Code:**

```python
# Example API implementation in Flask with a Redis cache
from flask import Flask, jsonify, request
import redis

app = Flask(__name__)
redis_client = redis.Redis(host='localhost', port=6379, db=0)

@app.route('/users/<int:user_id>/summary')
def get_user_summary(user_id):
    # Check cache first
    cached_data = redis_client.get(f'user_summary_{user_id}')
    if cached_data:
        return jsonify(cached_data.decode('utf-8'))

    # Fetch data from database and aggregate
    spending_data = Spending.query.filter_by(userId=user_id).all()
    summary = {}
    for spend in spending_data:
        if spend.categoryName not in summary:
            summary[spend.categoryName] = 0
        summary[spend.categoryName] += spend.amount

    # Store summary in cache
    redis_client.set(f'user_summary_{user_id}', jsonify(summary).data, ex=3600)  # Cache for 1 hour
    return jsonify(summary)
```

**Example input / output:**

```text
Request:
GET /users/1/summary

Response:
{
    "userId": 1,
    "categories": [
        { "categoryName": "food", "totalSpending": 500 },
        { "categoryName": "entertainment", "totalSpending": 300 }
    ]
}
```

**Data:**

```sql
CREATE TABLE Spending (
   id SERIAL PRIMARY KEY,
   userId INT REFERENCES Users(id),
   categoryName VARCHAR(50),
   amount DECIMAL(10, 2),
   timestamp TIMESTAMP
);
```

**Flow:**

1. Hit cache
2. Miss cache
3. User makes API request
4. Check cache for summary
5. Determine total spending percategory and return response
6. Fetch data from database
7. Aggregate data, store in cache

