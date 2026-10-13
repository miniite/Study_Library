---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd detect and break an event feedback loop where service A's event causes B to emit an event that A consumes and re-emits, under partial deploys where only some instances have the fix.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Read Your Writes in Event Systems](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Read_Your_Writes_in_Event_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem of detecting and breaking an event feedback loop in a partially deployed system, we need to ensure that services can correctly identify whether they are running with the latest version of code or not. This is crucial because in partial deploys, only some instances might have the fix for the feedback loop. Here’s how I would approach it:

First, let's define what an event feedback loop is: a scenario where service A emits an event that triggers service B to emit another event which service A consumes and then re-emits, creating an infinite cycle. To break this, we need a mechanism to identify whether the instance of each service has the fix or not.

**Comparison:**

```
Version	Behavior
1.0	Bypass
2.0	Process
```

**Code:**

```python
class Service:
    def __init__(self, version):
        self.version = version

    def handle_event(self, event):
        if self.should_bypass(event):
            return None  # Bypass processing for events that should be ignored

        return self.process_event(event)

    def process_event(self, event):
        # Implement the logic to emit a new event
        pass

    def should_bypass(self, event):
        # Check if the current version of this service should ignore the event
        pass
```

**Example input / output:**

```text
For instance, let's say Service A and B have a version `1.0` initially. After a partial deployment, some instances of Service A are updated to version `2.0`, which includes the fix for the feedback loop.

### Initial Setup
- **Service A (Version 1.0):** Emits an event.
- **Service B:** Consumes the event and emits another one back to Service A, causing a feedback loop.

### Partial Deployment
- Some instances of Service A are updated to version `2.0`.
- These instances have the fix for the feedback loop logic.

### Scenario
- A (Version 1.0) -> Event -> B -> New Event -> A (Version 1.0)
- A (Version 1.0) -> Event -> B -> New Event -> A (Version 2.0)

In this case, the new version of Service A should ignore the event that it just emitted.
```

**Data:**

```json
{
  "version": "2.0"
}
```

**Flow:**

1. Event
2. New Event
3. Event
4. New Event
5. Should Bypass?
6. Should Bypass?
7. Should Bypass?
8. Service A (Version 1.0)
9. (Service B)
10. Service A (Version 2.0)
11. [Bypass Processing]
12. [Process and Emit New Event]

