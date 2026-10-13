---
type: "practice"
difficulty: "Deep"
---

# You need to migrate a live 'notification preferences' model from a single boolean to per-channel-per-category granularity; how do you design the change to be backward-compatible and roll it out safely?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DTOs and Domain Models](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/DTOs_and_Domain_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a live 'notification preferences' model from a single boolean to per-channel-per-category granularity, we need to ensure both backward compatibility and safe rollout. The key is to design this change with clear separation of concerns, meticulous testing, and gradual migration strategies. This process involves redefining the data structure, updating existing code, and implementing a fallback mechanism.

Firstly, let's summarize what the question entails: We are dealing with an existing system where users have a single boolean value indicating their preference for receiving notifications (true or false). Our task is to extend this model so that each user can control preferences per-channel (e.g., email, push, SMS) and per-category (e.g., friend requests, billing, events).

#### Triaging the Question

The core of the question revolves around:

Backward Compatibility: Ensuring that existing data and code work seamlessly with the new model.
Safe Rollout: Minimizing disruptions during the transition period.

#### Assumptions
include:

The system already has a mechanism to handle user preferences.
The database schema can be modified as needed.
There is a way to deploy changes incrementally without causing downtime.
Step-by-Step Solution

Analyze Existing Model:

Current model: A single boolean has_notifications_enabled for each user.
New requirements: Per-channel-per-category preferences.

Design the New Data Structure:

Introduce a new table or structure within an existing one to store per-channel-per-category preferences.
Ensure that old data is migrated smoothly, e.g., if no preferences are set, default to the old boolean value.

Update Codebase:

Modify the code to read from both the old and new structures based on availability of newer data.
Implement fallback logic where possible (e.g., use the old boolean if no specific channel/category preference exists).

Implement Migrations Safely:

Use incremental deployment strategies like blue-green deployments or canary releases.
Monitor system performance closely during initial stages to catch any unexpected issues early.

Testing and Validation:

Thoroughly test the new model in staging environments before going live.
Validate edge cases, such as users who have no preferences configured yet still receive notifications based on legacy rules.

#### Closing Summary

By carefully designing the migration process, ensuring backward compatibility through fallback mechanisms, and deploying changes incrementally, we can safely roll out this feature without disrupting the existing user experience. The key is to balance between adopting new capabilities and maintaining stability during the transition period.

#### Common Mistakes/Follow-Ups

Forgetting to validate edge cases.
Not thoroughly testing in staging environments.
Ignoring user feedback or system performance metrics post-deployment, leading to unnoticed issues.

**Comparison:**

```
User ID	Email - Friend Requests	Push - Friend Requests
user123	true	false
Old Preference (Single Boolean)
True
New Preferences (Per-Channel-Per-Category)
Email: Friend Requests = true
Push: Friend Requests = false
```

**Code:**

```python
# Example of fallback logic in Python

def get_user_notification_preference(user_id, channel, category):
    # Check if new preferences exist for the user and specific context
    new_preferences = db.get_new_preferences(user_id, channel, category)
    if new_preferences is not None:
        return new_preferences

    # If no new preference exists, fall back to old boolean value
    old_preference = db.get_old_preference(user_id)
    default_preference = True  # Default value when no explicit setting found

    return old_preference or default_preference
```

**Example input / output:**

```text
Input (User ID, Channel, Category):
- user_id = 'user123'
- channel = 'email'
- category = 'friend_requests'

Output:
- True

Explanation:
New preferences exist and are used.
```

**Data:**

```json
{
  "new_preferences": {
    "user123": {
      "email": {
        "friend_requests": true,
        "billing": false,
        "events": true
      },
      "push": {
        "friend_requests": false,
        "billing": true,
        "events": false
      }
    }
  },
  "old_preference": {
    "user123": true
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Old Preference Handling
6. Set Old Preference as True
7. User Query
8. Check New Preferences Exist
9. New Preferences Found
10. Return New Preference Value
11. Check Old Boolean Preference
12. Old Boolean True?
13. Return Old Preference +Default
14. Return Default Preference
15. Store New Preferences
16. Inform User About Changes

