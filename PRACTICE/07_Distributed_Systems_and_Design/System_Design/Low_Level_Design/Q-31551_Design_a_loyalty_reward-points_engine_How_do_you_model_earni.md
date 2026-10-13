---
type: "practice"
difficulty: "Intermediate"
---

# Design a loyalty/reward-points engine. How do you model earning rules and redemption?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rule Engines and Pricing](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Rule_Engines_and_Pricing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a loyalty/reward-points engine, we need to focus on modeling both the earning and redemption of points. At an intermediate level, this involves creating a robust framework that ensures flexibility, maintainability, and scalability. We'll start by breaking down the core requirements and then dive into specific implementations.

The primary goals are:

To accurately track point earnings based on user activities.
To manage different earning rules for various activities (e.g., purchases, referrals).
To facilitate flexible redemption of points for rewards (e.g., discounts, gifts).

We will implement these requirements by designing the data models and algorithms from first principles.

**Comparison:**

```
Activity Type	Factor
purchase	1.0
referral	1.5
first_purchase	2.0
Reward Type	Points Required	Value
discount	100	$5
gift	500	free_tshirt
```

**Code:**

```python
# Example Python pseudo-code for point earning logic
class LoyaltyEngine:
    def __init__(self):
        self.user_points = {}

    def earn_points(self, user_id, activity_type, points):
        if user_id not in self.user_points:
            self.user_points[user_id] = 0
        # Apply any activity-specific rules here (e.g., bonus for referrals)
        adjusted_points = points * self.apply_activity_rule(activity_type, points)
        self.user_points[user_id] += adjusted_points

    def apply_activity_rule(self, activity_type, points):
        # Example rule: double points for first purchase
        if activity_type == "first_purchase":
            return 2.0
        return 1.0

# Example usage
loyalty_engine = LoyaltyEngine()
loyalty_engine.earn_points("user1", "purchase", 100)
```

**Example input / output:**

```text
Input:
- User ID: user1
- Activity Type: purchase
- Points Earned: 200

Output:
- Updated points for user1: 400 (double due to first_purchase rule)
```

**Data:**

```json
{
    "users": {
        "user1": {"points": 200, "activities": ["first_purchase", "second_referral"]}
    },
    "earn_rules": [
        {"activity_type": "purchase", "factor": 1.0},
        {"activity_type": "referral", "factor": 1.5},
        {"activity_type": "first_purchase", "factor": 2.0}
    ],
    "redeem_rules": [
        {"reward_type": "discount", "points_required": 100, "value": 5},
        {"reward_type": "gift", "points_required": 500, "value": "free_tshirt"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. User Activity
4. Earn Points
5. Apply Rules
6. Adjust Points
7. No Change
8. Redeposit Points
9. Rewrite Points to DB/Cache

