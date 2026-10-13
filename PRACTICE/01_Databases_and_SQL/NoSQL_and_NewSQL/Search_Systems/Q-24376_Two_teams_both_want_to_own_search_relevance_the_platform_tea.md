---
type: "practice"
difficulty: "Expert"
---

# Two teams both want to own 'search relevance' — the platform team and a product ML team. As principal, resolve this org-design conflict. Where do the responsibility boundaries and interfaces sit, and how do you prevent duplicated ranking stacks?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Search Relevance](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Relevance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the organizational design conflict between the platform team and the product ML team over 'search relevance,' we need to carefully define the scope of each team's responsibilities, establish clear interfaces, and ensure that there are no duplicated ranking stacks. The key is to balance domain expertise with shared goals while maintaining a modular system.

The core issue here involves defining the responsibility boundaries and ensuring seamless integration between teams. Typically, search platforms like Elasticsearch provide a flexible foundation for managing and optimizing text-based searches. However, relevance scoring can be highly context-dependent, often requiring custom machine learning (ML) models to capture nuanced user preferences or domain-specific knowledge.

#### Triage
What is Being Probed

The interviewer is probing:

Organizational Design: How responsibilities are allocated between different teams.
Interface Definition: Clear and well-defined communication channels.
Duplication Prevention: Ensuring that both teams do not independently develop similar components, which could lead to inefficiencies or inconsistencies.

#### Assumptions
Both the platform team and product ML team have relevant expertise in search and ML respectively.
The goal is to maintain a single, coherent user experience across all products.
There are no existing standardized solutions that can fully cover both aspects of relevance (indexing and ranking).

#### Solution
From First Principles
1. Define Core Responsibilities

Platform Team:

Indexing: Ensuring high-quality data ingestion and efficient indexing.
Query Processing: Basic query processing, highlighting, etc., but not complex ML-based refinements.

Product ML Team:

Relevance Modeling: Developing custom models to capture domain-specific relevance scores.
Ranking Optimization: Fine-tuning ranking algorithms based on user feedback and A/B testing results.
2. Establish Clear Interfaces
Data Ingestion: The platform team should provide clean, structured data for ingestion into Elasticsearch. This might include pre-processing steps like tokenization, stop-word removal, etc.
Model Deployment: The ML team can use APIs provided by the platform to deploy and manage their models without needing direct access to Elasticsearch internals.
3. Prevent Duplication
Shared Infrastructure: Use a common search backend (like Elasticsearch) where both teams can contribute. This ensures that customizations are done in a single place.
Modular Design: Encourage the development of modular components where possible, so each team works on distinct but interoperable parts.
4. Governance and Standards
Relevance Scoring API: Define clear APIs for relevance scoring that both teams can use to integrate their functionalities seamlessly.
Feedback Loops: Implement robust feedback mechanisms through A/B testing to continuously improve the search experience based on real-world usage data.

#### Closing Summary

By carefully defining responsibilities, establishing clear interfaces, and preventing duplication, we can ensure a cohesive and efficient search relevance system. The platform team will focus on foundational capabilities, while the ML team develops specialized models, ensuring that both teams contribute effectively without overlap.

#### Common Mistakes/Follow-Ups
Overlap in Responsibilities: Both teams might unintentionally work on similar areas if boundaries are not well-defined.
Lack of Feedback Mechanisms: Without proper feedback loops, the quality of search relevance could suffer over time.
Inefficiency due to Duplication: Redundant development can lead to wasted resources and inconsistent implementations.

To mitigate these risks, regular cross-team meetings, code reviews, and shared documentation are essential. Regularly reassessing the interfaces and responsibilities will also help in adapting to changing needs and technologies.

**Comparison:**

```
Team	Responsibilities
Platform	Indexing, Query Processing
ML	Relevance Modeling, Ranking Optimization
```

**Code:**

```python
# Example of a simple API for relevance scoring
def get_relevance_score(query, document):
    # This is a placeholder function; actual implementation would use ML models.
    return 0.75 * query_relevance + 0.25 * document_similarity

# Example usage
query = "best pizza recipe"
document = "How to make delicious pizza with homemade sauce."
score = get_relevance_score(query, document)
print(f"Relevance Score: {score}")
```

**Example input / output:**

```text
Input:
- Query: "best pizza recipe"
- Document: "How to make delicious pizza with homemade sauce."

Output:
- Relevance Score: 0.75 * query_relevance + 0.25 * document_similarity

Dry-run:
1. Query relevance = 0.8 (based on term frequency and query length)
2. Document similarity = 0.9 (based on cosine similarity with vectorized terms from the query)
3. Final Score = 0.75 * 0.8 + 0.25 * 0.9 = 0.66
```

**Data:**

```json
{
    "index": {
        "name": "product_reviews",
        "settings": {
            "analysis": {
                "filter": {
                    "stopwords_filter": {
                        "type": "stop",
                        "stopwords": ["the", "and", "or"]
                    }
                },
                "analyzer": {
                    "standard_analyzer": {
                        "tokenizer": "standard",
                        "filter": [
                            "lowercase",
                            "stopwords_filter"
                        ]
                    }
                }
            }
        }
    },
    "mappings": {
        "properties": {
            "title": {"type": "text", "analyzer": "standard_analyzer"},
            "content": {"type": "text", "analyzer": "standard_analyzer"}
        }
    }
}
```

**Flow:**

1. Platform Team
2. Indexing
3. Query Processing
4. ML Team
5. Relevance Modeling
6. Ranking Optimization
7. Shared Interface

