---
type: "practice"
difficulty: "Intermediate"
---

# Design a system to recommend similar products on a product page.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Order Tracking Design](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Order_Tracking_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that recommends similar products on a product page, we need to consider several key aspects such as data storage, recommendation algorithms, scalability, and performance optimizations. The core idea is to leverage user behavior data and product attributes to generate relevant recommendations efficiently.

The interviewer will be probing how you approach the problem from first principles, considering both algorithmic efficiency and system design trade-offs. Key assumptions include the availability of historical user interaction data (e.g., purchase history, ratings), a robust database system for storing products, and an API to fetch and display recommended products on the frontend.

Let’s break down the solution step-by-step:

Data Storage: Store product information in a scalable database that supports efficient querying.
Recommendation Algorithms: Implement algorithms based on collaborative filtering or content-based filtering.
Scalability Considerations: Ensure the system can handle a large number of products and users efficiently.
Performance Optimization: Optimize queries, caching mechanisms, and backend services to reduce latency.

First, let’s consider the data storage: We will use a relational database like PostgreSQL for storing product information along with metadata such as categories, tags, and historical user interactions. The schema might look something like this:

SQL
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    description TEXT,
    category_id INT REFERENCES categories(id),
    tags TEXT[]
);

CREATE TABLE user_interactions (
    user_id INT REFERENCES users(id),
    product_id INT REFERENCES products(id),
    interaction_type ENUM('view', 'purchase', 'rating'),
    timestamp TIMESTAMP
);

For recommendation algorithms, we can use a combination of collaborative filtering and content-based filtering:

Collaborative Filtering: Analyze the interactions between different users to find similar user behavior patterns.
Content-Based Filtering: Recommend products based on the attributes of the products and their similarity.

Now, let’s look at how these algorithms can be implemented:

User Similarity Calculation (Collaborative Filtering):

Compute cosine similarity or Jaccard index between users' interaction histories.
PYTHON
from sklearn.metrics.pairwise import cosine_similarity

# Example user interactions
user_a_interactions = [23, 45, 67]
user_b_interactions = [23, 89, 101]

similarity_matrix = cosine_similarity([user_a_interactions, user_b_interactions])

Product Similarity Calculation (Content-Based Filtering):

Use TF-IDF or cosine similarity to find similar products based on their attributes.
PYTHON
from sklearn.feature_extraction.text import TfidfVectorizer

# Example product descriptions
product_descriptions = ["product 1", "product 2", "product 3"]
vectorizer = TfidfVectorizer()
tfidf_matrix = vectorizer.fit_transform(product_descriptions)

similarity_scores = cosine_similarity(tfidf_matrix[0:1, :], tfidf_matrix)

To serve recommendations in real-time:

Use caching mechanisms like Redis to store frequently accessed data.
Implement an API endpoint that fetches recommended products based on user interactions and product attributes.

For example:

PYTHON
def get_recommendations(user_id):
    # Fetch top N similar users using collaborative filtering
    similar_users = find_similar_users(user_id)

    # Fetch products interacted by those users
    recommended_products = get_products_by_similar_users(similar_users)

    # Cache the result
    cache.set(f'recommendations_{user_id}', recommended_products, timeout=3600)

    return recommended_products

def find_similar_users(user_id):
    # Implement logic to find similar users based on interaction history
    pass

def get_products_by_similar_users(similar_users):
    # Fetch products interacted by those users and score them based on frequency
    pass

Finally, let’s summarize the key points:

Data Storage: Use a robust database schema for product and user interaction data.
Recommendation Algorithms: Implement collaborative and content-based filtering to generate recommendations.
Scalability: Optimize queries and use caching mechanisms to ensure efficient performance.
Performance Optimization: Use appropriate data structures and algorithms to handle large datasets.

#### Common mistakes
include overlooking the need for real-time updates, failing to consider cold start problems in recommendation systems, or not optimizing database queries adequately. Always validate assumptions with real-world examples to ensure robustness.

**Comparison:**

```
Field	Type
id	SERIAL
name	VARCHAR(255)
description	TEXT
category_id	INT
tags	TEXT[]
Field	Type
user_id	INT
product_id	INT
interaction_type	ENUM('view', 'purchase', 'rating')
timestamp	TIMESTAMP
```

**Code:**

```

from sklearn.metrics.pairwise import cosine_similarity

Example user interactions

user_a_interactions = [23, 45, 67] user_b_interactions = [23, 89, 101]

similarity_matrix = cosine_similarity([user_a_interactions, user_b_interactions])

CODE
Copy

```python
from sklearn.feature_extraction.text import TfidfVectorizer

# Example product descriptions
product_descriptions = ["product 1", "product 2", "product 3"]
vectorizer = TfidfVectorizer()
tfidf_matrix = vectorizer.fit_transform(product_descriptions)

similarity_scores = cosine_similarity(tfidf_matrix[0:1, :], tfidf_matrix)
```

**Example input / output:**

```text
def get_recommendations(user_id):
    # Fetch top N similar users using collaborative filtering
    similar_users = find_similar_users(user_id)

    # Fetch products interacted by those users
    recommended_products = get_products_by_similar_users(similar_users)

    # Cache the result
    cache.set(f'recommendations_{user_id}', recommended_products, timeout=3600)

    return recommended_products

def find_similar_users(user_id):
    # Implement logic to find similar users based on interaction history
    pass

def get_products_by_similar_users(similar_users):
    # Fetch products interacted by those users and score them based on frequency
    pass
```

**Data:**

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    description TEXT,
    category_id INT REFERENCES categories(id),
    tags TEXT[]
);

CREATE TABLE user_interactions (
    user_id INT REFERENCES users(id),
    product_id INT REFERENCES products(id),
    interaction_type ENUM('view', 'purchase', 'rating'),
    timestamp TIMESTAMP
);
```

**Flow:**

1. Yes
2. No
3. User Interaction
4. Is User Cache Available?
5. Cached Recommendations
6. Find Similar Users
7. Fetch Products Interacted bySimilar Users
8. Score Recommendationsbased on Frequency
9. Cache Recommendations
10. Return Recommendations

