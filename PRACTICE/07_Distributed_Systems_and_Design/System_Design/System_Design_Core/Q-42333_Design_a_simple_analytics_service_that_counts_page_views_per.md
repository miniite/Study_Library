---
type: "practice"
difficulty: "Basic"
---

# Design a simple analytics service that counts page views per article.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Order Tracking Design](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Order_Tracking_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a simple analytics service that counts page views per article, we need to focus on three main components: data collection, storage, and retrieval. This service will track how many times each article is viewed, which can be useful for content analysis, user engagement metrics, or improving the relevance of articles.

The question is probing our understanding of basic system design principles, including:

How to efficiently collect and process large volumes of data.
The choice between in-memory storage and persistent storage.
Scalability considerations and potential bottlenecks.
Error handling and fault tolerance mechanisms.

#### Assumptions
include that the service will be integrated into a web application where articles are accessed through URLs. We also assume that an article has a unique identifier, which is used to track page views.

Let’s start by defining our system from first principles:

Data Collection: Every time a user views an article, we need to increment its view count.
Storage: We need a storage mechanism where the article ID maps to the number of times it has been viewed.
Retrieval: The service should be able to retrieve and return the page view counts for any given set of articles.
Step-by-Step Solution

Choose Storage Mechanism:

For simplicity, we can use an in-memory cache like Redis or Memcached during development. This will allow us to quickly increment counters.
In a production environment, storing persistent data using a database such as PostgreSQL might be more appropriate for long-term retention and backup purposes.

Increment Counter:

When a user views an article, the service increments the corresponding counter in the chosen storage mechanism.
For example, if Redis is used, we can use INCR to increment a key associated with each article ID.

Retrieve Counters:

To retrieve page view counts for multiple articles, we need to fetch the values from our storage backend. In Redis, this would be done using MGET.

Error Handling and Fault Tolerance:

Implement error handling to manage cases where a user request fails due to network issues or service downtime.
Ensure high availability by replicating data across multiple nodes if using distributed systems.

Scalability Considerations:

As the number of articles grows, ensure that your storage solution can handle increased load without performance degradation.
Use caching strategies and read replicas to distribute reads efficiently.

#### Example
Implementation

Here’s a simple implementation using Redis as our in-memory cache:

PYTHON
import redis

# Initialize Redis client
redis_client = redis.Redis(host='localhost', port=6379, db=0)

def increment_view_count(article_id):
    # Increment view count for an article
    redis_client.incr(f'article:{article_id}')

def get_view_counts(article_ids):
    # Retrieve view counts for multiple articles
    keys = [f'article:{id}' for id in article_ids]
    results = redis_client.mget(keys)
    return {k.decode().split(':')[1]: int(v) if v else 0 for k, v in zip(keys, results)}

# Example usage
article_id = '12345'
increment_view_count(article_id)

view_counts = get_view_counts(['12345', '67890'])
print(view_counts)

#### Closing Summary

This simple analytics service provides a basic framework for tracking page views. By choosing the right storage mechanism, handling errors effectively, and ensuring scalability, we can build robust systems to support such needs in web applications.

#### Common mistakes
include not considering error conditions, underestimating the importance of fault tolerance, and failing to choose an appropriate data structure based on expected load.

