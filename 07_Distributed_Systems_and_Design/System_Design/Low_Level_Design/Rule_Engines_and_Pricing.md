---
id: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Rule_Engines_and_Pricing"
level: 4
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/Low_Level_Design"
tools: []
---

# Rule Engines and Pricing

**L4 Advanced** · [↑ Low Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Design a shopping-cart pricing pipeline with tax discounts and shipping. In what order do they apply?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31550_Design_a_shopping-cart_pricing_pipeline_with_tax_discounts_a.md>)
- [Intermediate] [Design a loyalty/reward-points engine. How do you model earning rules and redemption?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31551_Design_a_loyalty_reward-points_engine_How_do_you_model_earni.md>)
- [Intermediate] [Two engineers on your team model the same 'discount' concept differently in two services and it keeps causing bugs; how do you drive convergence on a shared domain model in a code review and what trade-offs do you weigh?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31601_Two_engineers_on_your_team_model_the_same_discount_concept_d.md>)
- [Deep] [How do you prevent two users from booking the same seat concurrently?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31444_How_do_you_prevent_two_users_from_booking_the_same_seat_conc.md>)
- [Deep] [Design a coupon and discount engine for an e-commerce cart.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31446_Design_a_coupon_and_discount_engine_for_an_e-commerce_cart.md>)
- [Deep] [Your Strategy-based pricing engine now has 30 strategy classes and finding the right one at runtime is a giant switch; how would you evolve the lookup and registration so adding a strategy needs zero edits to existing code?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31603_Your_Strategy-based_pricing_engine_now_has_30_strategy_class.md>)
- [Deep] [Requirements arrive mid-project that your booking system must support both seats and time-slots (movies and salons); how do you refactor the model to share the core without a fragile if/else on 'type'?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31611_Requirements_arrive_mid-project_that_your_booking_system_mus.md>)
- [Deep] [Your discount pipeline applies tax before discount in one code path and after in another, causing rupee mismatches customers complain about; how do you enforce a single canonical ordering in the design?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Q-31628_Your_discount_pipeline_applies_tax_before_discount_in_one_co.md>)

## Related concepts

- [Checkout System Design](<../High_Level_Design/Checkout_System_Design.md>) *(High Level Design)*
- [Rate Limiter Storage](<../System_Design_Core/Rate_Limiter_Storage/_index.md>) *(System Design Core)*
- [Payment Ledgers](<../High_Level_Design/Payment_Ledgers.md>) *(High Level Design)*

