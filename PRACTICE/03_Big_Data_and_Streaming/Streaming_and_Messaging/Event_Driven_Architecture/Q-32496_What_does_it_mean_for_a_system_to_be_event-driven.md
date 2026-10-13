---
type: "practice"
difficulty: "Basic"
---

# What does it mean for a system to be event-driven?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Choreography vs Orchestration](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Choreography_vs_Orchestration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an event-driven architecture, a system reacts to events (like user actions or data changes) by triggering corresponding handlers or processes. This model contrasts with traditional request-response architectures where a client sends a request and waits for a server response. The key idea here is that the flow of control moves from explicit requests to the occurrence of events.

To clarify what this means, we need to understand the components involved and how they interact:

Events: These are discrete occurrences such as clicks, data updates, or sensor readings.
Event Sources: Components generating these events, like a user interface (UI) or external sensors.
Event Listeners/Handlers: Processes that react to specific types of events. They process the event and take appropriate actions.

Now let's dive into how this architecture works in practice:

Event Generation: An event is generated when something interesting happens in the system. For instance, a form submission or a new data entry.
Event Propagation: This event is then sent to an event queue or directly to a listener based on its type and context.
Processing: The appropriate handler processes the event. This could be as simple as logging information or as complex as updating multiple services.
Response or Action: Based on the processing, some action might occur, such as sending another message or updating a database.

#### Common mistakes
in implementing an event-driven architecture include:

Overcomplicating the event flow without clear purpose.
Ignoring backpressure and handling failed events properly.
Not logging enough to diagnose issues when they arise.

**Code:**

```python
# Example of basic event handling in Python using a simple function
def handle_login_event(user_id):
    print(f"User {user_id} logged in.")
```

**Flow:**

1. Yes
2. No
3. Event Occurs
4. Is Event Type Known?
5. Dispatch to Handler
6. Handler Processes Event, e.g.,log or update state
7. Handle Unknown Event, e.g.,log error or ignore

