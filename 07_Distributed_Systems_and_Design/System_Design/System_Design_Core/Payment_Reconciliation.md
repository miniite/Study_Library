---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: []
---

# Payment Reconciliation

**L4 Advanced** · [↑ System Design Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Design a movie ticket booking system that prevents two users from booking the same seat.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42305_Design_a_movie_ticket_booking_system_that_prevents_two_users.md>)
- [Intermediate] [What happens if a payment succeeds at the gateway but your callback never arrives?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42355_What_happens_if_a_payment_succeeds_at_the_gateway_but_your_c.md>)
- [Deep] [How would you handle a money transfer between two accounts safely?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42285_How_would_you_handle_a_money_transfer_between_two_accounts_s.md>)
- [Deep] [How would you keep a wallet balance correct if two payment requests arrive at the same moment?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42308_How_would_you_keep_a_wallet_balance_correct_if_two_payment_r.md>)
- [Deep] [Design a system to reconcile your ledger against a payment gateway's settlement report daily. How do you handle missing callbacks, duplicates, and amounts that don't match?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42427_Design_a_system_to_reconcile_your_ledger_against_a_payment_g.md>)
- [Deep] [Under READ COMMITTED you see a lost-update anomaly on a counter. Walk me through when you'd move to REPEATABLE READ vs use SELECT FOR UPDATE vs an atomic UPDATE, and the cost of each.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42495_Under_READ_COMMITTED_you_see_a_lost-update_anomaly_on_a_coun.md>)
- [Deep] [Design a wallet ledger where balance can never go negative under concurrency. Compare SELECT FOR UPDATE, a conditional UPDATE ... WHERE balance >= amount, and event-sourced ledgers on throughput and auditability.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42525_Design_a_wallet_ledger_where_balance_can_never_go_negative_u.md>)
- [Expert] [How would you design a two-phase commit alternative (saga) for a multi-service booking that reserves a flight, a hotel, and a car — and what are your compensating actions when step 2 fails?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42409_How_would_you_design_a_two-phase_commit_alternative_saga_for.md>)

## Related concepts

- [Payment Ledgers](<../High_Level_Design/Payment_Ledgers.md>) *(High Level Design)*
- [Distributed Transactions and 2PC](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>) *(DBMS Concepts)*
- [Checkout System Design](<../High_Level_Design/Checkout_System_Design.md>) *(High Level Design)*

## References

- *System Design: The Big Archive* — Alex Xu (ByteByteGo) — Design a payment system

