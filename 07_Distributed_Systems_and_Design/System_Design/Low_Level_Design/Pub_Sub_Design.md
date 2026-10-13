---
id: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Pub_Sub_Design"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design"
tools: ["Apache_Kafka"]
---

# Pub Sub Design

**L4 Advanced** · [↑ Low Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Intermediate] [Design a hotel or a meeting-room booking system. Model rooms slots and bookings.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31489_Design_a_hotel_or_a_meeting-room_booking_system_Model_rooms_.md>)
- [Intermediate] [Design a notification-preference system so users can opt out per channel and per category.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31555_Design_a_notification-preference_system_so_users_can_opt_out.md>)
- [Intermediate] [Design a class model for a notification-batching/digest engine that groups N events into one email if they arrive within a window (e.g., '3 people liked your post'); how do you model the batching window and flush?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31778_Design_a_class_model_for_a_notification-batching_digest_engi.md>)
- [Deep] [Design a Pub/Sub message broker at the class level.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31455_Design_a_Pub_Sub_message_broker_at_the_class_level.md>)
- [Deep] [Design a chat-room mediator that must now scale to typing indicators, read receipts, and presence; how do you keep the mediator from becoming a god object as message types multiply?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31399_Design_a_chat-room_mediator_that_must_now_scale_to_typing_in.md>)
- [Deep] [You're the reviewer on a design that couples the domain layer to a specific message-broker's client; how do you articulate the long-term cost of that coupling to a team under deadline pressure, and what minimal seam do you insist on without over-engineering?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31806_You_re_the_reviewer_on_a_design_that_couples_the_domain_laye.md>)
- [Expert] [How do topics subscribers and delivery guarantees fit into your Pub/Sub design?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31456_How_do_topics_subscribers_and_delivery_guarantees_fit_into_y.md>)
- [Expert] [Design a fan-out-on-write vs fan-out-on-read decision for a follower feed at the class level; how do you make the write path pluggable so celebrity accounts can use a different strategy than normal users?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31699_Design_a_fan-out-on-write_vs_fan-out-on-read_decision_for_a_.md>)

## Related concepts

- [Fan-out Trade-offs](<../High_Level_Design/Fan-out_Trade-offs.md>) *(High Level Design)*
- [Real-time Counters](<../High_Level_Design/Real-time_Counters.md>) *(High Level Design)*
- [Notification System](<../System_Design_Core/Notification_System.md>) *(System Design Core)*

## References

- *System Design Interview Vol. 2* — Alex Xu, Sahn Lam — DESIGN A CHAT SYSTEM

