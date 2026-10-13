---
id: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Frontend_Graceful_Degradation"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/System_Design/System_Design_Core"
tools: []
---

# Frontend Graceful Degradation

**L5 Expert** · [↑ System Design Core](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [Design a system to store user preferences and settings that sync across devices.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42376_Design_a_system_to_store_user_preferences_and_settings_that_.md>)
- [Intermediate] [Design a frontend performance budget and the CI gates that enforce it. Which metrics (LCP, INP, bundle size) would you block a PR on, and how do you avoid death by false positives?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42433_Design_a_frontend_performance_budget_and_the_CI_gates_that_e.md>)
- [Intermediate] [Design state management for a large SPA with server state, URL state, and ephemeral UI state. When would you reach for a global store vs local state vs server-cache, and what's over-engineering here?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42489_Design_state_management_for_a_large_SPA_with_server_state_UR.md>)
- [Deep] [Design the architecture for a 'preview environment per pull request' that spins up frontend, backend, and a seeded database in under two minutes for 200 engineers. How do you keep costs sane and prevent these ephemeral envs from becoming a shadow production?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42176_Design_the_architecture_for_a_preview_environment_per_pull_r.md>)
- [Deep] [Walk me through how you'd run a pre-mortem for a critical new system before launch. What failure modes do principals surface that pass every code review, and how do you turn the pre-mortem into architectural changes?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42594_Walk_me_through_how_you_d_run_a_pre-mortem_for_a_critical_ne.md>)
- [Deep] [Design the end-to-end architecture for a 'time travel' debugging capability where support can replay exactly what a user saw and did (frontend state, network calls, backend responses) for a reported bug, without recording PII or ballooning storage. Where's the seam?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42712_Design_the_end-to-end_architecture_for_a_time_travel_debuggi.md>)
- [Deep] [Design the architecture for a system where a user-uploaded file is scanned, transformed, and made available across web and mobile, and the frontend must show accurate progress and a preview the moment it's safe, without exposing an unscanned or half-processed file. Where are the state transitions authoritative?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42736_Design_the_architecture_for_a_system_where_a_user-uploaded_f.md>)
- [Expert] [You're setting the strategy for how frontend state and backend cache stay coherent after a write: cache-aside invalidation, event-driven cache busting to the client, or server-push revalidation. Argue a default for a read-heavy social product and where you'd deviate.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Q-42191_You_re_setting_the_strategy_for_how_frontend_state_and_backe.md>)

## Related concepts

- [Micro-frontends](<../../High_Level_Design/Micro-frontends.md>) *(High Level Design)*
- [Real-time Counters](<../../High_Level_Design/Real-time_Counters.md>) *(High Level Design)*
- [Two-level Cache Design](<../../Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>) *(Low Level Design)*

