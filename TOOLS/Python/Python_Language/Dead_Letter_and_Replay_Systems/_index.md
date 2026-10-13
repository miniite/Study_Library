---
id: "TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems"
level: 4
type: "concept"
topic: "TOOLS/Python/Python_Language"
tools: ["Python"]
---

# Dead Letter and Replay Systems

**L4 Advanced** · [↑ Python Language](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Go deeper

- [Retries with Deadline Budgets](<Retries_with_Deadline_Budgets.md>)

## Implementations (tools)

- [Python](<../../_index.md>)

## Practice questions

- [Intermediate] [What is the 'finally' block guaranteed to do, and does it run even if the try block returns or raises?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36663_What_is_the_finally_block_guaranteed_to_do_and_does_it_run_e.md>)
- [Intermediate] [How would you schedule and run periodic jobs reliably in a Python service (APScheduler vs Celery beat vs external cron), and how do you avoid duplicate firing under multiple replicas?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36876_How_would_you_schedule_and_run_periodic_jobs_reliably_in_a_P.md>)
- [Intermediate] [Your agent occasionally loops forever calling the same tool with the same args. How would you detect and break the loop in Python, and what state would you track across turns?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37474_Your_agent_occasionally_loops_forever_calling_the_same_tool_.md>)
- [Deep] [Explain how you'd design graceful degradation in a Python API so that when a non-critical dependency (recommendations) is down, the core response still succeeds fast.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-36796_Explain_how_you_d_design_graceful_degradation_in_a_Python_AP.md>)
- [Deep] [Explain a real incident where a Python automation script you owned caused an outage or near-miss (e.g. a runaway delete loop). How did you contain it, and what guardrails did you add afterward?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37113_Explain_a_real_incident_where_a_Python_automation_script_you.md>)
- [Deep] [Your Python auto-remediation succeeded technically but the underlying issue recurred an hour later. How would you detect recurrence and escalate to a human instead of blindly re-remediating forever?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37777_Your_Python_auto-remediation_succeeded_technically_but_the_u.md>)
- [Deep] [How would you design a Python tool-calling agent loop that calls a function, feeds the result back, and decides whether to call again, while bounding the maximum number of tool iterations?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37473_How_would_you_design_a_Python_tool-calling_agent_loop_that_c.md>)
- [Expert] [How would you decide when to use an agent versus a fixed pipeline in Python for a task, and what signals tell you the added autonomy is worth the reliability and cost hit?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37525_How_would_you_decide_when_to_use_an_agent_versus_a_fixed_pip.md>)

## Related concepts

- [Compensating Transactions](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>) *(Low Level Design)*
- [Exactly Once Scheduled Jobs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Exactly_Once_Scheduled_Jobs.md>) *(Cloud Fundamentals)*
- [Reviewing AI Code for Performance Risks](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Performance_Risks.md>) *(AI Assisted Coding)*

