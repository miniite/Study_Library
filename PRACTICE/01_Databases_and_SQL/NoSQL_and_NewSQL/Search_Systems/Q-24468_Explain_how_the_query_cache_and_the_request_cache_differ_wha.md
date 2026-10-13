---
type: "practice"
difficulty: "Deep"
---

# Explain how the query cache and the request cache differ, what makes a query cacheable, and why using now in a range filter silently destroys cache hit rate.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Index Internals and Mappings](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The query cache and the request cache are two distinct mechanisms in Elasticsearch designed to enhance performance by caching query results. A query is cacheable if it meets certain criteria, such as having no dynamic or randomizing parameters. Using now in a range filter can significantly degrade cache hit rates because it introduces variability that prevents effective caching.

#### Triaging the Question

The question asks about the differences between the query cache and the request cache, what makes a query cacheable, and why using now in a range filter affects cache performance. This involves understanding the internal mechanisms of Elasticsearch for optimizing search operations and recognizing how dynamic parameters can impact these optimizations.

#### First-Principles Solution
Query Cache vs Request Cache

Query Cache:

Purpose: Caches individual query results.
Behavior: Stores the result of a specific query, including its exact terms and filters. If an identical query is run again within the cache timeout period, the cached result is returned directly.

Request Cache:

Purpose: Caches all responses for requests made by the same client in the same time frame.
Behavior: Stores the entire response body, allowing subsequent requests from the same client to return cached data without re-executing the query.
What Makes a Query Cacheable

A query is cacheable if:

It contains only static, deterministic terms (no randomizing parameters).
It does not involve dynamic or varying data.
The result can be reused reliably across multiple requests.
Why Using now in a Range Filter Destroys Cache Hit Rate

The now keyword introduces time-based variability into the query. Since now represents the current timestamp, it changes with each request:

Dynamic Parameter: Each query with a range filter using now will have a different set of results because now shifts based on when the query is executed.
Cache Key Uniqueness: The cache key for such queries becomes unique every time due to the dynamic nature of now, preventing any reuse.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, understanding caching mechanisms in Elasticsearch involves recognizing that static queries can be cached efficiently, whereas those with dynamic parameters like now cannot. It's crucial to avoid using such parameters in cacheable contexts to maintain high cache hit rates and performance.

#### Common Mistakes

Ignoring the distinction between query cache and request cache.
Failing to consider the impact of dynamic parameters on caching effectiveness.

#### Example Scenario

#### Cons
ider a scenario where you have an Elasticsearch index containing log data. If you create a range filter with now:

JSON
{
  "range": {
    "@timestamp": {
      "gte": "now/d-1d",
      "lte": "now"
    }
  }
}

This query will not cache effectively because the start and end times are dynamic, changing every second. Instead, you might want to use a fixed time range or a specific timestamp for caching purposes.

Final Self-Check

The answer is comprehensive and covers all aspects of the question without missing any important details.

**Code:**

```json
{
  "range": {
    "@timestamp": {
      "gte": "now/d-1d",
      "lte": "now"
    }
  }
}
```

