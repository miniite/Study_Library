---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Price_Feeds"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["Apache_Kafka", "Redis"]
---

# Real-time Price Feeds

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Deep] [During a flash sale, checkout traffic spikes 50x for 5 minutes — how do you design the system to absorb the surge without oversell or a total DB meltdown?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29608_During_a_flash_sale_checkout_traffic_spikes_50x_for_5_minute.md>)
- [Deep] [Design surge pricing for a ride-share app — how do you compute demand/supply ratios per geo-zone in near-real-time and roll out price changes consistently to clients?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29617_Design_surge_pricing_for_a_ride-share_app_how_do_you_compute.md>)
- [Deep] [Design a system to serve real-time stock price ticks to millions of concurrent web/mobile clients — how do you fan out updates efficiently without overwhelming clients?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29677_Design_a_system_to_serve_real-time_stock_price_ticks_to_mill.md>)
- [Deep] [Design near-real-time index updates so a price or stock change is searchable within seconds across a 100M-doc index — how do you propagate updates without full reindex and handle the read-during-refresh window?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29721_Design_near-real-time_index_updates_so_a_price_or_stock_chan.md>)
- [Deep] [Design budget pacing for an ad campaign so a daily budget is spent smoothly across the day without overspend — how do you distribute the budget across serving nodes and reconcile spend in near-real-time?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29726_Design_budget_pacing_for_an_ad_campaign_so_a_daily_budget_is.md>)
- [Deep] [Design frequency capping in ad serving so a user isn't shown the same ad more than N times in 24h across devices — where do you store the per-user counters and how do you keep the check under a few ms?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29729_Design_frequency_capping_in_ad_serving_so_a_user_isn_t_shown.md>)
- [Expert] [Architect capacity for a once-a-year event (say a mega-sale) that is 100x normal load for 10 minutes — how do you avoid paying for 100x capacity year-round while still guaranteeing the peak survives?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29802_Architect_capacity_for_a_once-a-year_event_say_a_mega-sale_t.md>)

## Related concepts

- [Org-wide Backpressure](<../System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [Rate Limiter Storage](<../System_Design_Core/Rate_Limiter_Storage/_index.md>) *(System Design Core)*
- [Clickstream Counting Pipeline](<../System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*

