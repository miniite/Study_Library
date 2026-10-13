---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd resolve a cross-team dispute where the platform team wants to enforce a canonical event schema and product teams want freedom. What's your decision framework and how do you avoid it recurring every quarter?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distinguishing Drift from Pipeline Breaks](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To resolve the cross-team dispute between the platform team, which wants to enforce a canonical event schema, and the product teams, who desire freedom in their data models, we need a balanced approach that aligns with our overall data architecture goals while accommodating the flexibility needed by the product teams. Our decision framework should involve several key steps: understanding the core requirements, defining clear roles and responsibilities, establishing a governance model, and implementing robust change management processes.

Firstly, it’s crucial to gather insights from both teams to understand their perspectives and constraints. The platform team is likely concerned about consistency, standardization, and ease of integration across systems. On the other hand, product teams might need unique event schemas to support specific business logic or user experiences without undue overhead.

**Comparison:**

```
Event	Fields	Platform Requirements	Product Team Needs
user_login	timestamp (datetime), user_id (int)	Yes	No
product_view	timestamp (datetime), user_id (int), product_id (str)	Yes	No
purchase	timestamp (datetime), user_id (int), product_id (str)	Yes	No
user_contact_request	timestamp (datetime), user_email (str), query_text (str)	No	Yes
customer_feedback	timestamp (datetime), user_email (str), feedback_text (str)	No	Yes
email_opened	timestamp (datetime), campaign_id (int), user_id (str)	No	Yes
link_clicked	timestamp (datetime), campaign_id (int), user_id (str)	No	Yes
```

**Code:**

```python
# Placeholder code to illustrate a simplified process for gathering input from both sides
def gather_input(platform_team, product_teams):
    platform_requirements = platform_team.get_consistency_requirements()
    product_needs = {team.name: team.get_event_schema() for team in product_teams}
    return platform_requirements, product_needs
```

**Example input / output:**

```text
Gathering input from both teams might yield the following:
- **Platform Requirements**: Standardized events like `user_login`, `product_view`, and `purchase` with consistent fields such as timestamp, user ID, and product ID.
- **Product Teams' Needs**:
  - Sales Team: Events include `user_contact_request` and `customer_feedback` with additional fields for user email and query text.
  - Marketing Team: Events like `email_opened` and `link_clicked` require campaign IDs and user IDs.

This output highlights the divergent needs, which must be reconciled to create a balanced solution that respects both teams' requirements.
```

**Data:**

```json
{
  "platform_requirements": {
    "events": ["user_login", "product_view", "purchase"],
    "fields": {"timestamp": "datetime", "user_id": "int", "product_id": "str"}
  },
  "product_needs": {
    "sales_team": {
      "events": ["user_contact_request", "customer_feedback"],
      "fields": {"timestamp": "datetime", "user_email": "str", "query_text": "str"}
    },
    "marketing_team": {
      "events": ["email_opened", "link_clicked"],
      "fields": {"timestamp": "datetime", "campaign_id": "int", "user_id": "str"}
    }
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. No
6. Yes
7. Start
8. Are the requirements clear?
9. Identify discrepancies
10. Can a compromise be found?
11. Define a hybrid schema
12. Implement and train teams
13. Revisit and refine requirements
14. Monitor and assess impact
15. Do the new requirements meetall needs?
16. Review process for futureconflicts

