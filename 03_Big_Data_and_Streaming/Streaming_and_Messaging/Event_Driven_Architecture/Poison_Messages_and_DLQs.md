---
id: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Poison_Messages_and_DLQs"
level: 4
type: "concept"
topic: "03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture"
tools: []
---

# Poison Messages and DLQs

**L4 Advanced** · [↑ Event Driven Architecture](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is a dead-letter queue and when would a message end up there?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32501_What_is_a_dead-letter_queue_and_when_would_a_message_end_up_.md>)
- [Intermediate] [How would you configure a dead-letter exchange in RabbitMQ and what routing do failed messages take?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32613_How_would_you_configure_a_dead-letter_exchange_in_RabbitMQ_a.md>)
- [Intermediate] [How do you distinguish a transient failure worth retrying from a permanent failure that should go straight to the DLQ?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32616_How_do_you_distinguish_a_transient_failure_worth_retrying_fr.md>)
- [Intermediate] [You run RabbitMQ and one queue has grown to 5 million messages, memory is spiking, and the broker is paging. What are your immediate mitigations and the longer-term design fixes?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32711_You_run_RabbitMQ_and_one_queue_has_grown_to_5_million_messag.md>)
- [Deep] [How do you handle a poison message that keeps failing and blocking the queue?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32534_How_do_you_handle_a_poison_message_that_keeps_failing_and_bl.md>)
- [Deep] [How do you handle poison messages that repeatedly crash the consumer process itself, not just fail logically?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32689_How_do_you_handle_poison_messages_that_repeatedly_crash_the_.md>)
- [Deep] [How would you design so that a poison message that crashes the consumer process (not just fails logically) can't halt the whole partition indefinitely?](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32764_How_would_you_design_so_that_a_poison_message_that_crashes_t.md>)
- [Deep] [A correlated failure took out both the primary and the DLQ because they shared a broker cluster, so failed messages had nowhere to go and were dropped. Explain why the DLQ must live in a different failure domain and how you'd architect that separation at scale.](<../../../PRACTICE/03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Q-32819_A_correlated_failure_took_out_both_the_primary_and_the_DLQ_b.md>)

## Related concepts

- [Dead Letter Queues and Outbox](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Dead_Letter_Queues_and_Outbox.md>) *(Distributed Systems Core)*
- [Idempotent Order APIs](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*
- [Read Your Writes with Replicas](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*

