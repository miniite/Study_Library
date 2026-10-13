---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between keyword and text fields and the multi-field pattern. Give a case where sorting or aggregating breaks because someone only mapped a field as text.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Index Internals and Mappings](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Elasticsearch, understanding the distinction between keyword and text fields is crucial for effective search and analysis operations. The multi-field pattern leverages both types to optimize query performance and data retrieval. However, improper mapping can lead to subtle issues, especially during sorting or aggregations.

Let's start with a brief summary: Keyword fields are primarily used for exact matches and facet aggregations, while text fields support full-text search operations. The multi-field pattern combines these two to provide both precision and flexibility. Misunderstanding this difference can result in common pitfalls such as incorrect sorting behavior.

#### Triaging the Question

The core of your question revolves around understanding:

Keyword vs Text Fields: Their usage, performance characteristics, and implications for search and analysis.
Multi-Field Pattern: How it leverages both field types to enhance functionality.
Common Pitfalls: Why sorting or aggregations might fail when only a text field is used.

#### First Principles Solution
Keyword vs Text Fields

Keyword Field:

Used for exact match queries and facet aggregations.
Indexing strategy focuses on storing the entire term exactly as entered.
Performance is faster because it’s stored verbatim without any processing or analysis.

Text Field:

Used for full-text search operations, such as querying content based on words or phrases.
Indexing involves tokenization (breaking text into tokens) and normalization (removing stop words, stemming, etc.).
Offers more flexibility in query terms but sacrifices some performance due to the additional processing.
Multi-Field Pattern

The multi-field pattern creates both keyword and text fields from a single source field. This allows you to index your data with full-text search capabilities while still retaining the ability to perform exact matches on the same data for aggregations and sorting.

For example, if you have an Elasticsearch mapping like this:

JSON
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      }
    }
  }
}

Here, title is a text field that supports full-text search. It also has a keyword sub-field named title.keyword.

Case Study: Sorting or Aggregating Fails

#### Cons
ider an e-commerce application where you have product titles stored in Elasticsearch as follows:

JSON
{
  "mappings": {
    "properties": {
      "title": { "type": "text" }
    }
  }
}

Suppose a query is issued to sort products by title:

HTTP
GET /products/_search
{
  "size": 5,
  "sort": [
    {"title": {"order": "asc"}}
  ]
}

Due to the nature of text fields, Elasticsearch performs tokenization and sorting based on these tokens. This can lead to unexpected results because the sort order is determined by the terms in the tokenized form, not the exact string.

For instance, if you have titles like:

"MacBook Pro"
"Macbook Air"
"MacBook 13"

The title field will be tokenized as follows:

"MacBook Pro" -> ["macbook", "pro"]
"Macbook Air" -> ["macbook", "air"]
"MacBook 13" -> ["macbook", "13"]

When sorting by these terms, you might get the following order:

"Macbook Air"
"MacBook Pro"
"MacBook 13"

This is because air comes before pro in lexicographical order.

However, if instead of using a text field, you had used both text and keyword fields:

JSON
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      }
    }
  }
}

You could sort by the title.keyword field, which ensures exact string matching:

HTTP
GET /products/_search
{
  "size": 5,
  "sort": [
    {"title.keyword": {"order": "asc"}}
  ]
}

This would give you the correct order of titles:

"MacBook Pro"
"Macbook Air"
"MacBook 13"

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, using only text fields can lead to unexpected sorting results due to tokenization. Ensuring that you map both text and keyword fields helps maintain the integrity of exact matches for sorting and aggregations.

#### Common mistakes

Failing to understand the difference between keyword and text.
Misusing text fields solely for sorting, leading to incorrect results.
Overlooking the need for multi-field patterns in complex search scenarios.

#### Follow-up
questions might include:

How does this impact performance?
What are other use cases where multi-field is beneficial?
Can you give an example of a more complex mapping scenario?

**Code:**

```json
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "title.keyword": { "type": "keyword" }
    }
  }
}
```

