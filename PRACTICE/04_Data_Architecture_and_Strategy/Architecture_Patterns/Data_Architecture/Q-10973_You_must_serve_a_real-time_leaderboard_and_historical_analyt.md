---
type: "practice"
difficulty: "Expert"
---

# You must serve a real-time leaderboard and historical analytics for a gaming platform with 50M concurrent players from the same event source. Architect the split between an in-memory/Redis serving tier, a real-time OLAP store, and the lakehouse, and where consistency between them is allowed to be loose.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect the data stack for serving a real-time leaderboard and historical analytics for a gaming platform with 50M concurrent players, we need to carefully split the data tiers into an in-memory/Redis serving tier, a real-time OLAP store (such as Apache Druid or StarRocks), and a lakehouse. The goal is to ensure that the system can handle high throughput and low latency while maintaining a balance between consistency, availability, and partition tolerance.

First, let's summarize the key components:

In-memory/Redis Serving Tier: Handles real-time leaderboard queries with sub-second response times.
Real-Time OLAP Store (e.g., Apache Druid or StarRocks): Aggregates data from the source in near-real time and serves it for more complex analytics.
Lakehouse (e.g., Delta Lake, Iceberg): Stores raw event data for long-term historical analysis and allows for efficient querying of large datasets.

Now, let's dive into the details:

#### Triage

The interviewer is probing:

How to balance real-time performance with historical analytics.
The trade-offs between different consistency models (eventual vs. strong).
The choice of technology stacks based on requirements.
Strategies for ensuring data integrity across multiple layers.

We need to make several assumptions:

The event source can handle high write throughput and low-latency reads.
We have a robust network infrastructure capable of handling the traffic between tiers.
Historical analytics queries are batch-based, allowing some delay in response times compared to real-time queries.

#### First Principles Solution

In-Memory/Redis Serving Tier:

Purpose: Real-time leaderboard updates and quick query responses.

#### Cons
istency Model: Eventual consistency since Redis is designed for low-latency operations with eventual updates.
Data Size: Small to medium-sized leaderboards, typically up to a few hundred entries per player.

Real-Time OLAP Store (e.g., Apache Druid or StarRocks):

Purpose: Aggregates and serves data for more complex queries like trending games, user behavior analysis.

#### Cons
istency Model: Strong consistency with minimal latency, allowing real-time query responses.
Data Size: Larger datasets than Redis, storing aggregated data from the source in a columnar format.

Lakehouse (e.g., Delta Lake, Iceberg):

Purpose: Stores raw event data for long-term historical analysis and efficient querying of large datasets.

#### Cons
istency Model: Eventual consistency but with robust ACID transactions to ensure data integrity during writes.
Data Size: Very large datasets, potentially petabytes in scale.
Detailed Solution

Event Source -> In-Memory/Redis Serving Tier:

The event source (e.g., player actions, game sessions) sends updates directly to Redis using a publish-subscribe pattern for low-latency updates.
Redis will maintain an up-to-date view of the leaderboard and other real-time metrics.

Event Source -> Real-Time OLAP Store:

Data from the event source is ingested into Apache Druid or StarRocks via a streaming pipeline (e.g., Apache Flink, Apache Spark Streaming).
Druid will aggregate data in near-real time and serve it for complex queries.
StarRocks can provide real-time analytics with lower latency than traditional batch processing.

Event Source -> Lakehouse:

Raw event data is written to the lakehouse (Delta Lake or Iceberg) using a structured format that supports efficient querying and ACID transactions.
The lakehouse allows for historical analysis of player behavior, game trends, and other insights by leveraging query optimization techniques.

#### Closing Summary

The architecture balances real-time performance with historical analytics needs. Redis ensures low-latency updates for leaderboards, while Apache Druid or StarRocks provide strong consistency for near-real-time analytics. Delta Lake/Iceberg handles long-term storage of raw data, ensuring data integrity and enabling efficient querying.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring the need for a robust network infrastructure between tiers.

#### Follow-Up
Discussing potential failure modes (e.g., network partitioning) and strategies to mitigate them.
Common Failure Modes
Network partitions can cause inconsistencies if not handled properly, especially when writing data to multiple layers.
Ensuring that writes are idempotent in the lakehouse is crucial to maintain data integrity during concurrent operations.

**Comparison:**

```
Tier	Data Size	Consistency Model	Latency
In-Memory/Redis	Small/Medium	Eventual consistency	<1 sec
Real-Time OLAP	Medium	Strong consistency	<100 ms
Lakehouse	Large	Eventually consistent	<5 sec
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Event Source] --> B[Redis Serving Tier];
    A --> C[Apache Druid/StarRocks Real-Time OLAP Store];
    A --> D[Lakehouse (Delta Lake/Iceberg)];
```

**Example input / output:**

```text
Input:
- Event from player login and game play

Output:
- Redis updates leaderboard with new score
- Druid ingests event and aggregates data for trending games
- Lakehouse stores raw event for future analysis
```

**Data:**

```json
{
  "event_source": {
    "player_actions": [
      {"timestamp": "2023-10-01T12:00:00Z", "player_id": 1, "action": "login"},
      {"timestamp": "2023-10-01T12:05:00Z", "player_id": 1, "action": "play_game"}
    ]
  },
  "redis_serving_tier": {
    "leaderboard": [
      {"player_id": 1, "score": 500, "timestamp": "2023-10-01T12:06:00Z"},
      {"player_id": 2, "score": 450, "timestamp": "2023-10-01T12:07:00Z"}
    ]
  },
  "real_time_olap_store": {
    "trending_games": [
      {"game_name": "Battle Royale", "plays": 5000, "timestamp": "2023-10-01T12:08:00Z"},
      {"game_name": "Arcade Classic", "plays": 4000, "timestamp": "2023-10-01T12:09:00Z"}
    ]
  },
  "lakehouse": {
    "raw_data": [
      {"player_id": 1, "action": "login", "timestamp": "2023-10-01T12:00:00Z"},
      {"player_id": 1, "action": "play_game", "timestamp": "2023-10-01T12:05:00Z"}
    ]
  }
}
```

**Flow:**

1. Event Source
2. Redis Serving Tier
3. Ledger Update (Leaderboard)
4. Apache Druid/StarRocksReal-Time OLAP Store
5. Trending Games Aggregation
6. Lakehouse (DeltaLake/Iceberg)
7. Raw Data Storage

