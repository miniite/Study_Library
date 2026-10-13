---
type: "practice"
difficulty: "Deep"
---

# How do you decide the right business-owner to sit in Phase 1 when the system serves both an internal ops org and external customers with conflicting goals? How does ATAM handle a system with two legitimate but opposed 'business owners'?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ ATAM for Build vs Buy](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/ATAM_for_Build_vs_Buy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When facing a scenario where a system must serve both internal operations (ops) and external customers, each with their own conflicting goals, the key is to leverage Architectural Trade-off Analysis Methodology (ATAM) to make an informed decision on which business owner should lead Phase 1. ATAM involves analyzing the system architecture through various perspectives such as performance, cost, security, scalability, maintainability, and risk. By thoroughly understanding both sets of goals and their impact on these architectural dimensions, we can determine a path forward that best serves our objectives.

The core idea is to identify the primary stakeholder whose strategic vision aligns most closely with the system's critical success factors for Phase 1. This involves:

Defining the business goals.
Identifying key stakeholders and their priorities.
Mapping these priorities onto architectural trade-offs.
Assessing risk and cost implications.
Choosing a path that balances the needs of both parties while minimizing overall risk.

**Comparison:**

```
Stakeholder	Internal Ops Goal	External Customer Goal
Cost	Minimize	Optimize
Performance	Stable	Scalable
Security	Enterprise	Consumer
Maintainability	Customizable	Standardized
```

**Flow:**

1. Define Business Goals
2. Identify Stakeholders
3. Mapped Priorities toArchitectural Trade-offs
4. Assess Risk and CostImplications
5. Choose Optimal Path
6. Implement Phase 1

