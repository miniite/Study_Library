---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["Redis", "Apache_Kafka"]
---

# Checkout System Design

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Deep] [In a notification service, how do you guarantee at-least-once delivery while deduplicating so a user never gets the same push twice after a retry?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29600_In_a_notification_service_how_do_you_guarantee_at-least-once.md>)
- [Deep] [Design a webhook delivery system that reliably delivers events to third-party endpoints — how do you handle retries, ordering, backoff, and slow/dead consumers?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29635_Design_a_webhook_delivery_system_that_reliably_delivers_even.md>)
- [Deep] [Design a quick-commerce (10-minute delivery) system's inventory and dark-store routing — how do you decide which store fulfills an order and reserve stock in real time?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29661_Design_a_quick-commerce_10-minute_delivery_system_s_inventor.md>)
- [Deep] [Design idempotent notification dispatch so that when the fanout worker crashes and its message is redelivered the user does not receive a duplicate push — where do you place the dedup key and what's its TTL?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29703_Design_idempotent_notification_dispatch_so_that_when_the_fan.md>)
- [Deep] [Design a seat/slot booking backend (concerts/flights) that holds a seat during the payment window and releases it on timeout at high contention — how do you avoid overbooking and hold leaks?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29751_Design_a_seat_slot_booking_backend_concerts_flights_that_hol.md>)
- [Deep] [You're the reviewer on a design proposing eventual consistency for a shopping-cart-to-order handoff — probe the specific double-order and lost-item failures and decide where you'd insist on a stronger guarantee and what it costs.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29948_You_re_the_reviewer_on_a_design_proposing_eventual_consisten.md>)
- [Expert] [How do you handle inventory reservation during checkout so two users can't buy the last unit — compare pessimistic locking, optimistic reservation, and eventual reconciliation.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29607_How_do_you_handle_inventory_reservation_during_checkout_so_t.md>)
- [Expert] [Design a saga for a multi-step order workflow spanning payment, inventory, and shipping services — orchestration vs choreography, and how do you handle compensating actions?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29646_Design_a_saga_for_a_multi-step_order_workflow_spanning_payme.md>)

## Related concepts

- [Idempotent Order APIs](<../System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*
- [Idempotent Consumers](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Idempotent_Consumers.md>) *(Event Driven Architecture)*
- [Compensating Transactions](<../Low_Level_Design/Compensating_Transactions/_index.md>) *(Low Level Design)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — Cassandra does not provide a mechanism to enforce transactions across table or keyspace boundaries

