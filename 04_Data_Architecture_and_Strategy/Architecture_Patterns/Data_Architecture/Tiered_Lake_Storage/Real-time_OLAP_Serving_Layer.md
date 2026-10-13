---
id: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Real-time_OLAP_Serving_Layer"
level: 5
type: "deep-dive"
topic: "04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture"
tools: ["Databricks", "Apache_Kafka", "AWS"]
---

# Real-time OLAP Serving Layer

**L5 Expert** · [↑ Data Architecture](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Databricks](<../../../../TOOLS/Databricks/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Explain when a real-time OLAP store (Druid/Pinot/ClickHouse) is worth running alongside your lakehouse versus when it's premature. What operational cost does the second store add?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11135_Explain_when_a_real-time_OLAP_store_Druid_Pinot_ClickHouse_i.md>)
- [Deep] [Your team dual-writes events to both the OLTP DB and Kafka, and they occasionally disagree. Explain the outbox pattern and how it would fix your reliability problem.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11083_Your_team_dual-writes_events_to_both_the_OLTP_DB_and_Kafka_a.md>)
- [Deep] [How do you decide whether real-time analytics belongs in a purpose-built OLAP store (Druid/Pinot/ClickHouse) vs your lakehouse's streaming tables? What determines the split and what's the cost of running both?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10619_How_do_you_decide_whether_real-time_analytics_belongs_in_a_p.md>)
- [Deep] [How do you architect the split between a time-series database (InfluxDB/TimescaleDB/Prometheus) and your general lakehouse for a manufacturing sensor estate? Where does downsampling and continuous-aggregate rollup belong, and when do you refuse to store raw high-frequency data in the lake at all?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10987_How_do_you_architect_the_split_between_a_time-series_databas.md>)
- [Expert] [How do you architect the boundary between the feature store's online serving layer and a purpose-built OLAP store (Pinot/Druid) when some ML features are also real-time analytics aggregates? When do you refuse to run both?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10755_How_do_you_architect_the_boundary_between_the_feature_store_.md>)
- [Expert] [How do you architect the primary-key and merge-on-read strategy for a lakehouse table fed by CDC upserts from a sharded OLTP source with 500M mutations/day? Where does merge-on-read's read amplification force you back to copy-on-write, and what's the compaction cadence?](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10947_How_do_you_architect_the_primary-key_and_merge-on-read_strat.md>)
- [Expert] [You must serve a real-time leaderboard and historical analytics for a gaming platform with 50M concurrent players from the same event source. Architect the split between an in-memory/Redis serving tier, a real-time OLAP store, and the lakehouse, and where consistency between them is allowed to be loose.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-10973_You_must_serve_a_real-time_leaderboard_and_historical_analyt.md>)
- [Expert] [How do you architect for 'query the past exactly as of a point in time' across a lakehouse where dimension tables are SCD Type 2 and fact tables are append-only? Walk me through the join logic and where naive as-of joins produce silently wrong historical reports.](<../../../../PRACTICE/04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Q-11002_How_do_you_architect_for_query_the_past_exactly_as_of_a_poin.md>)

## Related concepts

- [Lakehouse Fundamentals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>) *(Streaming Architecture)*

## References

- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Successful Design Patterns
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Architectural Patterns in Data Engineering

