---
type: "practice"
difficulty: "Deep"
---

# Make the build-vs-buy case for your Python agent orchestration: LangGraph/LlamaIndex vs an in-house framework. What criteria (control, lock-in, eval integration, hiring, roadmap risk) drive the decision at a 500-engineer org?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Safe User-authored Configuration](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this build-vs-buy scenario, we are evaluating whether to develop an in-house Python agent orchestration framework or use existing tools like LangGraph and LlamaIndex. For a 500-engineer organization, key criteria such as control, lock-in risk, evaluation integration, hiring considerations, and roadmap alignment will drive the decision.

We'll start by defining what each tool offers and then analyze these criteria in detail.

**Comparison:**

| Criteria              | In-House Framework           | LangGraph / LlamaIndex      |
|-----------------------|-----------------------------|----------------------------|
| Control               | Full control                | Limited by vendor policies  |
| Lock-in Risk          | High                         | Medium                      |
| Evaluation Integration| Easy integration            | Custom effort needed        |
| Hiring Considerations | High availability of skills | Learning curve, specialized |
| Roadmap Risk          | Aligned with internal needs | Vendor-driven priorities    |

**Code:**

```python
# Example code snippets for clarity (omitted here)
```

**Example input / output:**

```text
Input:
- Evaluate build-vs-buy for an in-house Python agent orchestration framework vs LangGraph/LlamaIndex.
- Consider a 500-engineer organization.

Output:
1. Control: In-house frameworks provide full control, while using third-party tools means relying on vendors' updates and policies.
2. Lock-in Risk: In-house solutions can be more flexible but require significant upfront investment to ensure compatibility with existing systems. Third-party tools come with vendor lock-in risks.
3. Evaluation Integration: In-house tools can be integrated seamlessly into existing evaluation pipelines, while third-party tools may require custom integration efforts.
4. Hiring Considerations: Availability of in-house expertise and candidates for Python frameworks is high; third-party tools might have a learning curve but offer specialized knowledge.
5. Roadmap Risk: Custom solutions are aligned with internal priorities but may not benefit from external advancements. Third-party tools can provide timely updates but pose risks if the vendor's roadmap diverges from organizational needs.
```

**Data:**

```json
{
  "criteria": [
    {
      "name": "Control",
      "description": "Degree of control over the solution."
    },
    {
      "name": "Lock-in Risk",
      "description": "Risk of becoming overly dependent on a specific tool or vendor."
    },
    {
      "name": "Evaluation Integration",
      "description": "Ease and efficiency in integrating with existing evaluation frameworks."
    },
    {
      "name": "Hiring Considerations",
      "description": "Availability of skilled candidates who are familiar with the technology."
    },
    {
      "name": "Roadmap Risk",
      "description": "Risk associated with future changes or roadmaps of the chosen solution."
    }
  ]
}
```

**Flow:**

1. Define Criteria
2. In-House Framework
3. Control
4. Lock-in Risk
5. Evaluation Integration
6. Hiring Considerations
7. Roadmap Risk
8. Full control
9. High
10. Easy integration
11. High availability of skills
12. Vendor-driven priorities

