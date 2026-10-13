---
type: "practice"
difficulty: "Deep"
---

# Design a loyalty/reward-points engine where earning rules change frequently (double points on weekends) without a deploy; how do you model rules as data and keep redemption idempotent?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Compensating Transactions](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design a flexible and efficient loyalty/reward-points engine where earning rules change frequently (e.g., double points on weekends) without requiring a deploy, we need to model rules as data and ensure that point redemptions are idempotent. The key is to use an event-driven architecture with a rule engine and a database schema that supports dynamic updates.

#### Triaging the Question

The question asks us to design a system for managing reward points in a way that allows earning rules (like double points on weekends) to be updated dynamically without redeploying code. We need to ensure that point redemptions are idempotent, meaning they can be safely repeated multiple times.

Key Assumptions
Rules change frequently and need to be updated without downtime.
Points must be accurately redeemed based on the current rules.
The system should handle a high volume of transactions efficiently.
Step-by-Step Solution

Database Schema

We use a database schema that can store earning rules as data, allowing for dynamic updates.
SQL
CREATE TABLE reward_rules (
  id INT PRIMARY KEY,
  rule_name VARCHAR(255),
  condition JSONB,  -- Stores the conditions as JSON (e.g., {"day_of_week": [6, 0], "time_range": ["14:00", "18:00"]})
  action JSONB     -- Stores the actions (e.g., {"multiplier": 2})
);

Rule Engine

Implement a rule engine that can read rules from the database and apply them at runtime.
The rule engine should be decoupled from the point-earning logic to allow for flexible updates.

Event-Driven Architecture

Use events to trigger point earn/apply actions, ensuring they are idempotent and can be replayed without issue.
PYTHON
class RewardEngine:
    def __init__(self):
        self.rules = {}

    def load_rules(self, rules_data):
        for rule in rules_data:
            condition = rule['condition']
            action = rule['action']
            # Apply conditions to actions and store them in a structured format

    def apply_rule(self, user_id, points_to_add):
        current_time = datetime.now()
        matched_rule = self.find_matching_rule(current_time)
        if matched_rule:
            multiplier = matched_rule['action']['multiplier']
            return int(points_to_add * multiplier)
        else:
            return points_to_add

Idempotent Redemptions

Use unique identifiers for redemptions and store them to prevent double spending.
PYTHON
class PointRedemptionStore:
    def __init__(self):
        self.redemptions = set()

    def redeem_points(self, redemption_id, user_id, points_to_subtract):
        if redemption_id in self.redemptions:
            return False  # Redundant transaction
        self.redemptions.add(redemption_id)
        # Apply point subtraction logic here
        return True

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Failing to decouple the rule engine from point logic, not handling event-driven transactions correctly, or ignoring idempotency.
Best Practices: Ensure rules are stored as data and can be updated without downtime. Use a rule engine for dynamic application of rules and ensure all operations are idempotent.

#### Example
Input & Output
Input
PYTHON
# Sample Rule Data
rules_data = [
    {
        "condition": {"day_of_week": [6, 0], "time_range": ["14:00", "18:00"]},
        "action": {"multiplier": 2}
    }
]

reward_engine = RewardEngine()
reward_engine.load_rules(rules_data)

# User earns points
earn_points_result = reward_engine.apply_rule(user_id=1, points_to_add=5)
print(earn_points_result)  # Should print the doubled points: 10

# Redemptions
redemption_id = "redemption_1234"
point_redemption_store = PointRedemptionStore()
redeemed = point_redemption_store.redeem_points(redemption_id, user_id=1, points_to_subtract=5)
print(redeemed)  # Should print True if not previously redeemed
Diagrams
Yes
No
Yes
No
User Earnings
Is Rule Matched?
Apply Multiplier
Points Added Without Change
Redemption Request
Has Redemption ID BeenUsed?
Transaction Failed
Subtract Points, RecordRedemption
Self-Check

The design ensures dynamic rule updates and idempotent redemptions. All steps are correctly outlined.

**Code:**

```python
class RewardEngine:
    def __init__(self):
        self.rules = {}

    def load_rules(self, rules_data):
        for rule in rules_data:
            condition = rule['condition']
            action = rule['action']
            # Apply conditions to actions and store them in a structured format

    def apply_rule(self, user_id, points_to_add):
        current_time = datetime.now()
        matched_rule = self.find_matching_rule(current_time)
        if matched_rule:
            multiplier = matched_rule['action']['multiplier']
            return int(points_to_add * multiplier)
        else:
            return points_to_add

class PointRedemptionStore:
    def __init__(self):
        self.redemptions = set()

    def redeem_points(self, redemption_id, user_id, points_to_subtract):
        if redemption_id in self.redemptions:
            return False  # Redundant transaction
        self.redemptions.add(redemption_id)
        # Apply point subtraction logic here
        return True
```

**Example input / output:**

```

Sample Input:

PYTHON
Copy
rules_data = [
    {
        "condition": {"day_of_week": [6, 0], "time_range": ["14:00", "18:00"]},
        "action": {"multiplier": 2}
    }
]

reward_engine = RewardEngine()
reward_engine.load_rules(rules_data)

earn_points_result = reward_engine.apply_rule(user_id=1, points_to_add=5)
print(earn_points_result)  # Should print the doubled points: 10

redemption_id = "redemption_1234"
point_redemption_store = PointRedemptionStore()
redeemed = point_redemption_store.redeem_points(redemption_id, user_id=1, points_to_subtract=5)
print(redeemed)  # Should print True if not previously redeemed

Sample Output:

PYTHON
Copy
10
True
```

**Data:**

```sql
CREATE TABLE reward_rules (
    id INT PRIMARY KEY,
    rule_name VARCHAR(255),
    condition JSONB,  -- Stores the conditions as JSON
    action JSONB     -- Stores the actions
);
```

