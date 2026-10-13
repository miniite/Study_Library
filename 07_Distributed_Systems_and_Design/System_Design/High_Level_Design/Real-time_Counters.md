---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Counters"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["Redis"]
---

# Real-time Counters

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Design the analytics dashboard for a URL shortener that shows real-time click counts per link — how do you keep the frontend counters fresh without hammering the DB on every redirect?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29585_Design_the_analytics_dashboard_for_a_URL_shortener_that_show.md>)
- [Intermediate] [Design the rider's live-map frontend showing the assigned driver moving toward them — how do you smooth GPS jitter, handle dropped location updates, and keep the ETA in sync?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29618_Design_the_rider_s_live-map_frontend_showing_the_assigned_dr.md>)
- [Deep] [Design a like/reaction counter that stays accurate for a post going viral (10K reactions/sec) without a hot-row problem — how do you aggregate and serve the count?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29639_Design_a_like_reaction_counter_that_stays_accurate_for_a_pos.md>)
- [Deep] [Design fraud-scoring in the payment authorization path that must add under 30ms of latency — how do you fetch features rules and velocity counters synchronously and fall back safely if the scorer times out?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29691_Design_fraud-scoring_in_the_payment_authorization_path_that_.md>)
- [Deep] [Design a feature-flag evaluation service that returns flag values in under 1ms server-side with gradual percentage rollouts and targeting rules — how do you distribute config and keep evaluation deterministic per user?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29757_Design_a_feature-flag_evaluation_service_that_returns_flag_v.md>)
- [Deep] [Design a feature store that serves ML features online at low latency and computes them offline consistently — how do you avoid train/serve skew and keep online features fresh from streaming updates?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29775_Design_a_feature_store_that_serves_ML_features_online_at_low.md>)
- [Expert] [Design a real-time-vs-batch reconciliation architecture for user-facing counters (views/likes) where the fast approximate number and the accurate nightly number disagree — how do you reconcile without users noticing jumps?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29855_Design_a_real-time-vs-batch_reconciliation_architecture_for_.md>)
- [Expert] [Architect a multiplayer game backend for a real-time session-based game (100 players per match) — how do you design authoritative simulation state reconciliation with lag compensation matchmaking and anti-cheat server-side validation?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29955_Architect_a_multiplayer_game_backend_for_a_real-time_session.md>)

## Related concepts

- [Frontend Graceful Degradation](<../System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>) *(System Design Core)*
- [Event-time Watermarks](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>) *(Streaming Architecture)*
- [Clickstream Counting Pipeline](<../System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*

