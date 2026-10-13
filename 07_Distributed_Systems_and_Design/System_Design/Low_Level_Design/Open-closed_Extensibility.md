---
id: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Open-closed_Extensibility"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design"
tools: []
---

# Open-closed Extensibility

**L4 Advanced** · [↑ Low Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Design a currency-conversion module that plugs in different rate providers. Which pattern decouples them?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31554_Design_a_currency-conversion_module_that_plugs_in_different_.md>)
- [Intermediate] [Probe: if I ask you to add a new payment method tomorrow, which files change in your design and why so few?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31593_Probe_if_I_ask_you_to_add_a_new_payment_method_tomorrow_whic.md>)
- [Intermediate] [You must add a new 'partial refund' capability to a payments module without touching the hot 'capture' path; how do you find the extension seam and prove you didn't regress the capture flow?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31635_You_must_add_a_new_partial_refund_capability_to_a_payments_m.md>)
- [Deep] [Design a rate-card or a tax calculation module that varies by region. Which pattern?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31496_Design_a_rate-card_or_a_tax_calculation_module_that_varies_b.md>)
- [Deep] [How would you design a payment gateway abstraction so swapping Razorpay for Stripe in one region is a config change, and what seams do you build to keep provider-specific quirks from leaking into the domain?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31608_How_would_you_design_a_payment_gateway_abstraction_so_swappi.md>)
- [Deep] [How would you unit-test an LLD that depends on the current time, random shuffling, and a payment gateway; walk through the seams (clock, RNG, gateway) you'd design in so tests are deterministic.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31663_How_would_you_unit-test_an_LLD_that_depends_on_the_current_t.md>)
- [Deep] [A VP challenges you to justify why the org should invest in a shared money/currency type when "floats have worked fine"; walk through the concrete production failures you'd cite and how you'd pilot the change to prove it out before a broad rollout.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31816_A_VP_challenges_you_to_justify_why_the_org_should_invest_in_.md>)
- [Expert] [You must set the org standard for money-rounding on multi-way splits (a 3-way split of 100.00) so no team loses or invents a paisa; design the money type's split contract, where the remainder goes deterministically, and how you'd enforce it across payments code.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31880_You_must_set_the_org_standard_for_money-rounding_on_multi-wa.md>)

## Related concepts

- [Payment Ledgers](<../High_Level_Design/Payment_Ledgers.md>) *(High Level Design)*
- [Org-wide Lineage and PII Tracking](<../System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>) *(System Design Core)*
- [Backpressure and Load Shedding](<../High_Level_Design/Backpressure_and_Load_Shedding.md>) *(High Level Design)*

