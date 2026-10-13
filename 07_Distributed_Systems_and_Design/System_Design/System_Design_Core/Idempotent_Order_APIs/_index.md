---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: ["Apache_Kafka", "Redis"]
---

# Idempotent Order APIs

**L4 Advanced** · [↑ System Design Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [API Composition and BFF](<API_Composition_and_BFF.md>)

## Implementations (tools)

- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Redis](<../../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [Design a basic email service that queues and sends bulk marketing emails.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42320_Design_a_basic_email_service_that_queues_and_sends_bulk_mark.md>)
- [Deep] [How would you handle inventory and prevent overselling in an e-commerce checkout?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42157_How_would_you_handle_inventory_and_prevent_overselling_in_an.md>)
- [Deep] [How would you retry a failed webhook without spamming the receiver?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42271_How_would_you_retry_a_failed_webhook_without_spamming_the_re.md>)
- [Deep] [Design a file upload feature that supports resuming interrupted large uploads.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42358_Design_a_file_upload_feature_that_supports_resuming_interrup.md>)
- [Deep] [You inherited a synchronous checkout flow that calls payment, inventory, and email inline. Prod times out under load. How would you redesign it to be async and idempotent while keeping the user-facing latency low?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42395_You_inherited_a_synchronous_checkout_flow_that_calls_payment.md>)
- [Deep] [Design a job scheduler that guarantees at-most-once execution of a task at a precise future time across a fleet, tolerating machine crashes. How do you avoid two workers grabbing the same job?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42420_Design_a_job_scheduler_that_guarantees_at-most-once_executio.md>)
- [Deep] [Design a system to safely process refunds where the same refund request may arrive multiple times and the gateway is eventually consistent. How do you guarantee at-most-one refund?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42463_Design_a_system_to_safely_process_refunds_where_the_same_ref.md>)
- [Deep] [A poison message keeps crashing your consumer and blocking the partition. Design a dead-letter-queue flow with retry limits, and explain how you'd replay a fixed batch of DLQ messages safely.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42509_A_poison_message_keeps_crashing_your_consumer_and_blocking_t.md>)

## Related concepts

- [Checkout System Design](<../../High_Level_Design/Checkout_System_Design.md>) *(High Level Design)*
- [Compensating Transactions](<../../Low_Level_Design/Compensating_Transactions/_index.md>) *(Low Level Design)*
- [Exactly Once Scheduled Jobs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Exactly_Once_Scheduled_Jobs.md>) *(Cloud Fundamentals)*

## References

- *System Design: The Big Archive* — Alex Xu (ByteByteGo) — Avoid double charge

