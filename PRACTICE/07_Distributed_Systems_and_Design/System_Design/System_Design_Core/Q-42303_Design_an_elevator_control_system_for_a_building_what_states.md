---
type: "practice"
difficulty: "Intermediate"
---

# Design an elevator control system for a building — what states and events matter?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an elevator control system for a building, we need to identify the key states and events that drive the behavior of the elevators. This problem involves understanding the core functionalities required and ensuring a robust and efficient system. Let's break down the critical components and their interactions.

#### Triaging the Question

The question asks us to design an elevator control system, which means we need to consider both the high-level architecture and the detailed behaviors of the elevators within the building. The key elements are:

States: These represent the internal conditions of the elevator.
Events: These are external or internal actions that can change the state of the elevator.
Core Solution

The core solution involves defining a clear set of states for each elevator and understanding how these states transition based on various events. The goal is to create an efficient system that minimizes wait times and maximizes throughput while ensuring safety.

States and Events
States
Idle: The elevator is not serving any passengers.
Serving Call: The elevator is in motion, responding to a call from the current floor or another floor.
Stopping at Floor: The elevator has arrived at a requested floor.
Opening Doors: The doors are opening on the specified floor.
Closing Doors: The doors are closing after passengers have exited or entered.
Events
Call for Service: A passenger presses the call button to request an elevator.
Request Floor Change: Passengers enter and exit the elevator, changing its target floors.
Floor Arrival: The elevator has arrived at a requested floor.
Door Opening/Closing: The doors open or close as passengers board/detrain.

#### First-Principles Solution

To design this system from first principles:

Identify Core Entities: Elevator, Floor, Call, and Passenger.
Define State Transitions:
From Idle to Serving Call when a call is made.
From Serving Call to Stopping at Floor as the target floor is reached.
From Stopping at Floor to Opening Doors and then Closing Doors.
Implement Event Handling:
Handle new calls by prioritizing based on factors like current direction, distance, etc.
Update passenger information in real-time.

#### Implementation
Example

Here's a simplified pseudocode outline for the elevator control system:

PYTHON
class Elevator:
    def __init__(self):
        self.state = "Idle"
        self.floor = 0
        self.target_floors = []

    def handle_call(self, floor):
        if self.state == "Idle":
            self.target_floors.append(floor)
            self.state = "Serving Call"
        elif self.state in ["Stopping at Floor", "Opening Doors", "Closing Doors"]:
            # Handle inter-floor calls here
            pass

    def update_state(self):
        if self.state == "Serving Call":
            # Move elevator to the nearest floor and stop
            pass
        elif self.state in ["Opening Doors", "Closing Doors"]:
            # Open/close doors, then change state
            pass

# Example Usage
elevator = Elevator()
elevator.handle_call(10)
elevator.update_state()

#### Closing Summary
+ Common Mistakes/Follow-Ups

The elevator control system involves defining clear states and events that govern the behavior of the elevators. The key is to ensure efficient state transitions while handling various calls and passenger movements. Common mistakes include:

Overcomplicating the state machine.
Failing to consider edge cases like multiple simultaneous calls or emergency stops.

For follow-ups, we could discuss load balancing across multiple elevators, optimizing routes for different building layouts, or incorporating predictive algorithms to improve efficiency based on historical data.

**Comparison:**

```
State	Events
Idle	New call
Serving	Floor arrival, door open
Stopping	Door close
```

**Code:**

```python
class Elevator:
    def __init__(self):
        self.state = "Idle"
        self.floor = 0
        self.target_floors = []

    def handle_call(self, floor):
        if self.state == "Idle":
            self.target_floors.append(floor)
            self.state = "Serving Call"
        elif self.state in ["Stopping at Floor", "Opening Doors", "Closing Doors"]:
            # Handle inter-floor calls here
            pass

    def update_state(self):
        if self.state == "Serving Call":
            # Move elevator to the nearest floor and stop
            pass
        elif self.state in ["Opening Doors", "Closing Doors"]:
            # Open/close doors, then change state
            pass

# Example Usage
elevator = Elevator()
elevator.handle_call(10)
elevator.update_state()
```

**Example input / output:**

```text
Input:
- Elevator handles a call at floor 10.

Output:
- Elevator transitions from "Idle" to "Serving Call", with target floors updated.
- Elevator updates its state as it moves and stops at the relevant floors, opens/closes doors accordingly.
```

**Data:**

```json
{
  "elevator": {
    "state": "Idle",
    "floor": 0,
    "target_floors": [10]
  }
}
```

**Flow:**

1. Idle
2. Serving Call
3. Stopping at Floor
4. Opening Doors
5. Closing Doors

