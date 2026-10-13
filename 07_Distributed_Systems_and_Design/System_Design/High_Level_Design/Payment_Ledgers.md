---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Payment_Ledgers"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: []
---

# Payment Ledgers

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Deep] [Explain the CAP theorem in practical terms for a payment system — where do you choose consistency over availability, and what does that cost you during a partition?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29653_Explain_the_CAP_theorem_in_practical_terms_for_a_payment_sys.md>)
- [Deep] [Your payment capture service must guarantee a card is authorized then captured exactly once even if the auth response is lost on the network — walk through the state machine reconciliation query and timeout handling you'd build.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29684_Your_payment_capture_service_must_guarantee_a_card_is_author.md>)
- [Deep] [Your payments DB has a hot row on the merchant's balance account during peak sale hours causing lock contention — how do you redesign the balance update to remove the hotspot while keeping the running balance queryable?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29690_Your_payments_DB_has_a_hot_row_on_the_merchant_s_balance_acc.md>)
- [Deep] [Design the architecture for a BNPL (buy-now-pay-later) product spanning eligibility real-time credit decisioning disbursement and repayment schedules — how do you keep the credit-decision path fast and the money-movement ledger correct across all three legs?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29903_Design_the_architecture_for_a_BNPL_buy-now-pay-later_product.md>)
- [Expert] [Design a payment ledger / double-entry accounting system — how do you guarantee balances are always consistent and every transaction is auditable and replayable?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29633_Design_a_payment_ledger_double-entry_accounting_system_how_d.md>)
- [Expert] [Design the reconciliation subsystem for a payments platform that matches internal ledger entries against daily PSP settlement files — how do you detect missing captured-but-not-settled and settled-but-not-recorded transactions and drive them to resolution?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29683_Design_the_reconciliation_subsystem_for_a_payments_platform_.md>)
- [Expert] [Design a double-entry ledger service where every write must be atomic across debit and credit accounts at 20K TPS — how do you shard accounts while still keeping cross-account transfers consistent?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29689_Design_a_double-entry_ledger_service_where_every_write_must_.md>)
- [Expert] [Adjudicate consistency-vs-availability for a global inventory system where overselling costs money but downtime costs more — where exactly do you place the CAP line per operation and why?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29799_Adjudicate_consistency-vs-availability_for_a_global_inventor.md>)

## Related concepts

- [Payment Reconciliation](<../System_Design_Core/Payment_Reconciliation.md>) *(System Design Core)*
- [Multi-tenant Migrations](<../System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*
- [Idempotent Order APIs](<../System_Design_Core/Idempotent_Order_APIs/_index.md>) *(System Design Core)*

## References

- *System Design Big Archive (2023)* — ByteByteGo — Cloud Native Landscape
- *System Design: The Big Archive* — Alex Xu (ByteByteGo) — Stock exchange design

