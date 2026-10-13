---
type: "practice"
difficulty: "Deep"
---

# A search returns relevant docs but ranked terribly. Walk me through how you'd debug scoring — explain output, boosting, function_score, and constant_score — to fix relevance without breaking recall.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Relevance](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Relevance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a poorly-ranked search result in Elasticsearch, we need to understand and fine-tune various aspects of the scoring mechanism. In this context, "output" refers to the scored results returned by Elasticsearch; "boosting," "function_score," and "constant_score" are key concepts that influence how documents are ranked. By analyzing and adjusting these components, we can improve relevance without compromising recall.

#### Triaging the Question

The question is essentially asking you to debug a poorly-ranked search result in an Elasticsearch index. The interviewer wants to understand your approach to:

Understanding the current ranking output: What scores do documents receive, and why?
Explaining and utilizing boosting mechanisms: How can we adjust field weights or document attributes to influence scoring?
Implementing function_score queries: How can complex scoring logic be applied using this query type?
Using constant_score queries: How does it fit into the overall strategy for ranking?

#### Assumptions
The search query is correctly formulated.
The index and mapping are well-defined.
There's no underlying issue with data quality or indexing.

#### First Principles Solution
Step 1: Understand the Current Scoring Output

First, we need to understand what scores documents are receiving. We can use the _mget API to fetch the top results along with their raw scores and stored fields:

JSON
GET /index_name/_mget
{
  "docs": [
    {"_id": "<top_id>"},
    {"_id": "<second_top_id>"},
    ...
  ]
}

This will give us insights into the documents that are currently ranked high.

Step 2: Exploring Boosting Mechanisms

Boosting is a way to influence the score of specific fields or documents. This can be achieved by setting field boosting in the query, using ^ notation:

JSON
GET /index_name/_search
{
  "query": {
    "multi_match": {
      "query": "search_query",
      "fields": ["title^2", "content"],
      "type": "most_fields"
    }
  }
}

Here, the title field is boosted by a factor of 2. Understanding which fields are most relevant and boosting them can improve relevance.

Step 3: Implementing Function Score Queries

Function score queries allow us to apply complex scoring logic on top of a base query:

JSON
GET /index_name/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "title": "search_query"
        }
      },
      "functions": [
        {
          "script_score": {
            "script": "doc['popularity'].value * Math.log(doc['age'].value + 1)"
          }
        }
      ],
      "score_mode": "multiply",
      "boost_mode": "replace"
    }
  }
}

In this example, popularity and age are used to boost relevance. score_mode determines how scores from the base query and functions combine.

Step 4: Using Constant Score Queries

#### Cons
tant score queries can be useful when we want a certain filter to always apply at a constant factor:

JSON
GET /index_name/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      },
      "boost": 1.5
    }
  }
}

This ensures that documents with status: active are always scored higher, but the boost factor is configurable.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, debugging Elasticsearch scoring involves understanding current outputs, leveraging boosting mechanisms to influence field weights, utilizing function_score queries for complex logic, and ensuring that constant score filters maintain necessary conditions. Common mistakes include over-reliance on a single approach without considering the broader context, ignoring data quality issues, or failing to validate changes through A/B testing.

#### Follow-ups
might involve:

Discussing how caching strategies can impact performance.
Exploring alternative ranking mechanisms like BM25 or TF-IDF.
Addressing issues related to schema design and indexing strategies.

**Comparison:**

```
Document ID	Title	Content	Popularity	Age	Status
<top_id>	Top Title	Top Content	5	3	active
<second_top_id>	Second Title	Second Content	4	2	inactive
```

**Code:**

```

GET /index_name/_mget { "docs": [ {"_id": "<top_id>"}, {"_id": "<second_top_id>"}, ... ] }

CODE
Copy

```json
GET /index_name/_search
{
  "query": {
    "multi_match": {
      "query": "search_query",
      "fields": ["title^2", "content"],
      "type": "most_fields"
    }
  }
}
JSON
Copy
GET /index_name/_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "title": "search_query"
        }
      },
      "functions": [
        {
          "script_score": {
            "script": "doc['popularity'].value * Math.log(doc['age'].value + 1)"
          }
        }
      ],
      "score_mode": "multiply",
      "boost_mode": "replace"
    }
  }
}
JSON
Copy
GET /index_name/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      },
      "boost": 1.5
    }
  }
}
```

**Example input / output:**

```

GET /index_name/_search { "query": { "multi_match": { "query": "search_query", "fields": ["title^2", "content"], "type": "most_fields" } } }

CODE
Copy

Input:
- Query: `search_query`
- Fields: `title` (weight: 2), `content`

Output:
```json
[
  {"_id": "<top_id>", "_score": 10.5},
  {"_id": "<second_top_id>", "_score": 8.4}
]
```

**Data:**

```json
[
  {"_id": "<top_id>", "_source": {"title": "Top Title", "content": "Top Content", "popularity": 5, "age": 3, "status": "active"}},
  {"_id": "<second_top_id>", "_source": {"title": "Second Title", "content": "Second Content", "popularity": 4, "age": 2}}
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the search query correctlyformulated?
5. Understand current rankingoutput using _mget API
6. Analyze field weights andboost relevant fields
7. Implement function_scorequeries for complex scoringlogic
8. Use constant_score filters tomaintain necessary conditions
9. Validate changes through A/Btesting
10. End
11. Correct the query formulation

