---
type: "practice"
difficulty: "Deep"
---

# You must set the org guideline for when to use the null-object pattern versus optional/absent; a team returns a real 'guest user' object to avoid null checks and it silently grants permissions; how do you draw the line so null-object never hides a real absence?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Tenant Isolation in Repositories](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When deciding between using the null-object pattern versus optional/absent values, the key is to ensure that null objects are used only when there's an actual, meaningful "default" state, not just as a way to avoid null checks. The goal is to make it clear and obvious when real absence of data or states should be represented by null rather than a default value.

#### Triaging the Question

The question revolves around understanding when to use the null-object pattern in contrast to using optional/absent values. The core issue here is avoiding hidden errors that null objects can introduce, where a "guest user" object might silently grant permissions without raising any issues. Our objective is to clearly distinguish between genuine absences and default states.

#### Assumptions
Null Objects vs Optional Values: Null objects are used when there's an actual meaningful state or behavior (e.g., the guest user has certain implicit permissions). Absent values, on the other hand, might simply mean that no value is present.
Team Practices: The team currently returns a real 'guest user' object to avoid null checks and it silently grants permissions. This needs to be reevaluated.
Step-by-Step Solution from First Principles:

Identify Genuine Absences:

Determine when null truly means "nothing" or "no value". For example, if a user is not logged in, returning null should indicate an absence of a user object.

Define Default States:

Use null objects for default states that have specific behaviors or roles. A 'guest user' could be seen as such—a state with certain implicit permissions but still distinct from a truly anonymous user.

Avoid Hiding Errors:

Ensure that using null does not mask real errors or absences. If a method should return an object, it should either return the expected type or throw an exception if no valid data is available.

Documentation and Naming Conventions:

Clearly document when null objects are used versus when optional values or exceptions are preferred.
Use descriptive names for variables to convey their purpose (e.g., nullGuestUser vs userOrNull).

Code Review and Testing:

Implement a code review process that specifically checks for the misuse of null objects in place of absent values.
Include tests that cover both valid and invalid scenarios to ensure robustness.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Common Mistake: Using null as a substitute for absent values, leading to silent failures or misleading behavior. For example, returning a 'guest user' object might make it hard to trace issues where permissions are unexpectedly granted.

#### Follow-Up
Questions:

How do you manage performance and memory trade-offs when dealing with null objects versus optional types?
What about edge cases like partial data? How should your guidelines address situations where only some fields of an entity are missing?

#### Closing Summary

By carefully distinguishing between genuine absences (null) and default states (null objects), we can ensure that our code is clear, robust, and free from hidden errors. This requires a thorough understanding of the domain and careful documentation to prevent misuse.

**Comparison:**

```
Scenario	Null Object Usage?	Optional Value Used?
Genuine Absence (User Not Logged In)	No	Yes
Default State (Guest User)	Yes	No
```

**Code:**

```
MERMAID
Copy
graph TD
    A[Define null object when there's an actual state] --> B{Is it a genuine absence?}
    B -- Yes --> C1[Use null]
    B -- No --> C2[Use optional/absent value]
    C1 --> D[Ensure clear documentation for usage]
    C2 --> E[Integrate with error handling]
```

**Example input / output:**

```text
def get_user_session() -> UserSession:
    if session.get('is_guest'):
        return GuestUserSession()
    else:
        return RealUserSession(session['id'], session['name'])

# Test case
session = {'is_guest': True}
user_session = get_user_session()
print(user_session.name)  # Output: "Guest User"
```

**Data:**

```yaml
# Example schema to illustrate usage

user:
  type: object
  properties:
    id: integer
    name: string
    isGuestUser: boolean
```

**Flow:**

1. Yes
2. No
3. Check Session for Guest User
4. Is guest?
5. Return Guest User Object
6. Throw Authentication Error

