---
type: "practice"
difficulty: "Intermediate"
---

# Design a state machine for an order that can go placed to shipped to delivered or cancelled.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ State Pattern](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/State_Pattern.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: We will design a state machine using finite state automata concepts, where each state represents an order's status (Placed, Shipped, Delivered, Cancelled). The state transitions and actions performed upon transition are the core of our design.

#### Triaging the question
The interviewer is probing your understanding of state machines, event handling in software systems, and how to model complex workflows. We will assume that this system needs to handle order states and provide appropriate events or actions at each transition between states.

Step-by-step solution from first principles:

Define States: Identify all possible states an order can be in: Placed, Shipped, Delivered, Cancelled.
Define Events/Actions: For each state transition, define the events that could cause a change (e.g., "ship_order" for transitioning from Placed to Shipped).
Determine Transitions and Actions: Define which actions should be taken when an event occurs.

Let's break down these steps:

Define States:

Placed: The order has been created and is awaiting processing.
Shipped: The order has been sent out for delivery.
Delivered: The order has been successfully delivered to the customer.
Cancelled: The order has been terminated before completion.

Define Events/Actions:

Placed -> Shipped: ship_order
Shipped -> Delivered: deliver_order
Shipped -> Cancelled: cancel_order

Determine Transitions and Actions:

When ship_order is triggered, the system transitions from Placed to Shipped.
When deliver_order is triggered, the system transitions from Shipped to Delivered.
When cancel_order is triggered, the system transitions from Shipped or Placed to Cancelled.

Now, let's implement this state machine using a simple object-oriented approach in Python. We will use an enum for states and a class to handle events and transitions.

PYTHON
from enum import Enum

class OrderState(Enum):
    PLACED = 1
    SHIPPED = 2
    DELIVERED = 3
    CANCELLED = 4

class OrderStateMachine:
    def __init__(self, state=OrderState.PLACED):
        self.state = state

    def process_event(self, event):
        if self.state == OrderState.PLACED and event == "ship_order":
            self.state = OrderState.SHIPPED
            print("Order shipped.")
        elif self.state == OrderState.SHIPPED and event == "deliver_order":
            self.state = OrderState.DELIVERED
            print("Order delivered.")
        elif (self.state == OrderState.PLACED or self.state == OrderState.SHIPPED) and event == "cancel_order":
            self.state = OrderState.CANCELLED
            print("Order cancelled.")
        else:
            print(f"Invalid transition: {event} from state {self.state}")

# Example usage
order = OrderStateMachine()
order.process_event("ship_order")  # Order shipped.
order.process_event("deliver_order")  # Order delivered.
order.process_event("cancel_order")  # Order cancelled.

#### Closing summary
This design ensures that the order can transition between states based on specific events, and appropriate actions are taken during transitions. The use of enums makes state management clearer and more maintainable.

#### Common mistakes/follow-ups

Forgetting to handle invalid event transitions (e.g., trying to ship an already delivered order).
Not accounting for edge cases like a cancelled order before shipping.
Overcomplicating the state machine implementation without clear need.

