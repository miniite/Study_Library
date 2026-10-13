---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd store token-level streaming latency (time-to-first-token, inter-token latency) per request in SQL and query the models with the worst TTFT under load.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Experiment Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Experiment_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively store and analyze token-level streaming latency data such as time-to-first-token (TTFT) and inter-token latency, we need to design a robust SQL schema that can handle high concurrency and provide fast query performance under load. The key challenge is to ensure that the storage mechanism is scalable and efficient, allowing us to quickly identify models with the worst TTFT during periods of heavy traffic.

#### Triaging the Question

The question asks for an SQL solution to store token-level streaming latency data per request and to efficiently query models based on their TTFT under load. The interviewer is probing our understanding of:

Schema design for high concurrency.
Indexing strategies.
Query optimization techniques.

#### Trade-offs
between storage complexity and query performance.

#### First Principles Solution
Schema Design:
Use a requests table to store request metadata such as model ID, timestamp, and other relevant fields.
Create separate tables for TTFT and inter-token latency data to avoid bloating the main request table.
Indexing Strategy:
Index the model_id column in all latency-related tables to enable fast querying by model.
Query Optimization:
Leverage window functions and indexing to quickly retrieve models with the worst TTFT during peak times.
Detailed Solution
Schema Design:
SQL
CREATE TABLE requests (
    id BIGSERIAL PRIMARY KEY,
    model_id INT NOT NULL REFERENCES models(id),
    request_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE ttft (
    id BIGSERIAL PRIMARY KEY,
    request_id BIGINT NOT NULL REFERENCES requests(id),
    first_token_time TIMESTAMP NOT NULL
);

CREATE TABLE inter_tokens (
    id BIGSERIAL PRIMARY KEY,
    request_id BIGINT NOT NULL REFERENCES requests(id),
    token_index INT NOT NULL,
    latency TIMESTAMP NOT NULL -- difference between consecutive tokens
);
Indexing Strategy:
SQL
CREATE INDEX idx_requests_model_id ON requests(model_id);
CREATE INDEX idx_ttft_request_id ON ttft(request_id);
CREATE INDEX idx_inter_tokens_request_id ON inter_tokens(request_id);
Query Optimization:

To find models with the worst TTFT under load, we can use a window function to rank the models based on their average TTFT over a sliding window of requests.

SQL
WITH ttft_aggregates AS (
    SELECT
        r.model_id,
        AVG(t.first_token_time - r.request_time) OVER (PARTITION BY r.model_id ORDER BY r.request_time ROWS BETWEEN 10 PRECEDING AND CURRENT ROW) as avg_ttft
    FROM requests r
    JOIN ttft t ON r.id = t.request_id
)
SELECT model_id, AVG(avg_ttft) as avg_worst_ttft
FROM ttft_aggregates
GROUP BY model_id
ORDER BY avg_worst_ttft DESC;

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that we can efficiently store and query token-level streaming latency data. By using a combination of a well-designed schema, appropriate indexing, and advanced SQL techniques, we minimize storage complexity while maximizing query performance. Common mistakes include:

Over-indexing, which can lead to performance degradation.
Failing to consider the trade-offs between real-time and historical data retrieval.

#### Follow-up
questions might involve discussing how to handle edge cases, such as dealing with stale data or optimizing for different load patterns.

