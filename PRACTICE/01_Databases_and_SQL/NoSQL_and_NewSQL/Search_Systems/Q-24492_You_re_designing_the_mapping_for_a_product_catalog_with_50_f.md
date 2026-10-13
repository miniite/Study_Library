---
type: "practice"
difficulty: "Intermediate"
---

# You're designing the mapping for a product catalog with 50 fields but only 8 are ever searched. How do you keep the mapping lean, and what's the cost of leaving everything as default dynamic mapping?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Platform Scaling](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Platform_Scaling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing the mapping for a product catalog with 50 fields where only 8 are actually searched, the key is to strike a balance between keeping the mapping lean and ensuring optimal performance and storage efficiency. This involves carefully defining the non-searchable fields as not_analyzed or using dynamic mappings appropriately.

TL;DR: Define custom mappings for the searchable fields while leaving the others with default dynamic settings or explicitly not analyzed, thus reducing index size and improving query speed without compromising functionality.

#### Triage
Understanding the Question

The interviewer is probing your understanding of Elasticsearch mapping, performance optimization, and trade-offs between storage efficiency and search flexibility. You need to explain how to minimize unnecessary overhead while ensuring that the most critical fields are properly indexed for search queries.

#### Assumptions
The product catalog contains 50 fields, but only 8 are used in searches. Elasticsearch is being used as the backend for this application.

#### First-Principles Solution
Identify Searchable Fields: First, identify which of the 50 fields will be part of your search criteria.
Define Custom Mappings: For these searchable fields, define explicit mappings to ensure they are correctly indexed and analyzed for text searching purposes.
Use Default Dynamic Mappings Carefully: For non-searchable fields, use default dynamic mappings or mark them as not_analyzed to avoid unnecessary storage and processing overhead.
Step-by-Step Solution

Identify Searchable Fields:

Let's assume the 8 searchable fields are: name, description, price, category, brand, color, size, and stockLevel.

Define Custom Mappings for Searchable Fields:

JSON
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "description": { "type": "text", "analyzer": "standard" },
      "price": { "type": "float" },
      "category": { "type": "keyword" },  // Not analyzed
      "brand": { "type": "text" },
      "color": { "type": "keyword" },     // Not analyzed
      "size": { "type": "text" },
      "stockLevel": { "type": "integer" }
    }
  }
}

Use Default Dynamic Mappings for Non-Searchable Fields:

For the remaining 42 fields, use the default dynamic mappings.
JSON
{
  "mappings": {
    "_doc": {
      "dynamic_templates": [
        { "not_analyzed_fields": { "match": "*", "mapping": { "type": "keyword", "ignore_above": 256 } } }
      ]
    }
  }
}

Evaluate and Optimize:

After defining the mappings, test performance metrics such as query response time, index size, and overall resource usage.
Use Elasticsearch’s _cat APIs or Kibana to monitor these metrics.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By carefully defining custom mappings for only the fields that are searched, we reduce unnecessary storage overhead and improve search performance. Default dynamic mappings can be useful but should be used judiciously to avoid bloat in the index size. Common mistakes include overusing not_analyzed or default dynamic settings for all fields, leading to poor query performance or increased storage costs.

#### Common Mistakes
Overgeneralization: Using too many not_analyzed mappings can lead to slower queries and more complex maintenance.
Inconsistent Mappings: Failing to define specific mappings for searchable fields can result in suboptimal search results or missing data during analysis.

#### Follow-Ups
How do you handle updates to the schema?
What about edge cases where new searchable fields are added frequently?
How do you balance storage and performance in a growing dataset?

**Code:**

```json
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "description": { "type": "text", "analyzer": "standard" },
      "price": { "type": "float" },
      "category": { "type": "keyword" },  // Not analyzed
      "brand": { "type": "text" },
      "color": { "keyword": {} },         // Not analyzed
      "size": { "type": "text" },
      "stockLevel": { "type": "integer" }
    }
  }
}

{
  "mappings": {
    "_doc": {
      "dynamic_templates": [
        { "not_analyzed_fields": { "match": "*", "mapping": { "type": "keyword", "ignore_above": 256 } } }
      ]
    }
  }
}
```

