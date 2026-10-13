---
type: "practice"
difficulty: "Deep"
---

# You want to suppress child alerts when a parent dependency is already firing (alert-storm noise reduction). How would you express this dependency-aware suppression in SQL using a self-join or NOT EXISTS against the topology?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gaps and Islands](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Gaps_and_Islands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need to suppress child alerts when their parent dependencies are already firing, we can use SQL to leverage a self-join or NOT EXISTS clause based on the dependency topology. This is crucial for reducing alert storms and improving noise reduction in AIOps systems.

The core idea here is that if a parent alert is firing, no child of this parent should trigger an alert unless it has independent conditions that warrant one. We can model this using a hierarchical structure where each alert node may have dependent nodes (children). Our SQL query will ensure that for any given alert, we check if its parent is already firing before allowing the alert to be processed.

Let’s assume our alerts table (alerts) has columns like id, parent_id, and is_firing. The parent_id column indicates which alert depends on another (if it's null or not applicable, consider it as a root node).

First, we'll outline the problem: If an alert with id = A is firing (is_firing = 1), any of its children should not be allowed to fire. To achieve this, we need to identify all alerts that are children of A, and ensure none of them can fire if A is already firing.

Here's how you would express this using a self-join:

SQL
SELECT child_alert.id
FROM alerts AS parent_alert
JOIN alerts AS child_alert ON child_alert.parent_id = parent_alert.id
WHERE parent_alert.is_firing = 1 AND child_alert.is_firing = 0;

Alternatively, we can use NOT EXISTS to achieve the same result:

SQL
SELECT id
FROM alerts
WHERE is_firing = 1 AND NOT EXISTS (
    SELECT 1 FROM alerts AS child WHERE child.parent_id = alerts.id AND child.is_firing = 1
);

Both queries effectively suppress firing children if their parent is already firing.

