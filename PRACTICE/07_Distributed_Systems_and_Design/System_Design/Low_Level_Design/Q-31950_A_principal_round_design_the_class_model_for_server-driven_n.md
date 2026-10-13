---
type: "practice"
difficulty: "Deep"
---

# A principal round: design the class model for 'server-driven navigation' where the backend controls the next screen in a flow; how do you keep the client a dumb renderer while still supporting client-side validation and deep links without the backend knowing UI internals?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Tenant Isolation in Repositories](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this design, we are tasked with creating a class model for 'server-driven navigation' where the server controls the next screen in a flow. The goal is to keep the client application as a "dumb renderer" while enabling client-side validation and deep links without exposing UI internals to the backend. This approach ensures that the client remains lightweight, secure, and maintainable.

#### Triage

The key aspects of this problem are:

Server-Driven Navigation: The server dictates the next screen in the flow.
Client as a Dumb Renderer: The client should only render data without any complex logic.
Client-Side Validation: Implement validation rules on the client-side to ensure user inputs meet criteria before sending requests to the backend.
Deep Links: Support for deep linking to specific parts of an application using URLs.

#### First-Principles Solution

To achieve this, we need a robust architecture that separates concerns effectively while maintaining security and performance. We will design the following components:

Navigation Service Layer (Backend): Manages navigation logic and communicates with the client.
Presentation Layer (Client): Renders screens based on server instructions but performs validation locally.
Data Model: Structures data for efficient transfer between layers.
Navigation Service Layer

The backend will handle:

Storing and managing navigation states.
Generating URLs for deep linking.
Handling requests from the client to determine the next screen.
NavigationService
Client
Presentation Layer

The client will be responsible for:

Rendering screens based on instructions received from the backend.
Performing validation before sending data to the backend.
Handling deep links by processing incoming URLs.
Client
Screen A
Validator
Data Model

We define a simple JSON structure for navigating between screens:

JSON
{
  "id": "screen1",
  "name": "Login Screen",
  "next": {
    "id": "screen2",
    "name": "Home Screen"
  }
}
Detailed Implementation
Backend (NavigationService)

This service will handle the logic for determining the next screen and validating inputs.

PYTHON
class NavigationService:
    def __init__(self):
        self.navigation_states = {}

    def add_navigation_state(self, state_id, next_screen):
        self.navigation_states[state_id] = next_screen

    def get_next_screen(self, current_state):
        return self.navigation_states.get(current_state)

navigation_service = NavigationService()
Client (Presentation Layer)

The client will receive instructions from the backend and perform local validation.

JAVASCRIPT
class Client {

#### cons
tructor(navigationService) {
        this.navigationService = navigationService;
        this.validator = new Validator();
    }

    async handleNavigationRequest(stateId, userInput) {

#### cons
t nextScreen = await this.navigationService.get_next_screen(stateId);
        if (nextScreen && this.validator.validate(userInput)) {
            // Perform client-side rendering or redirection
        } else {

#### cons
ole.error("Validation failed or no valid state found");
        }
    }
}
Validator

The validator ensures that user inputs meet the required criteria.

JAVASCRIPT
class Validator {
    validate(input) {
        // Implement validation logic here
        return true; // Placeholder
    }
}

#### Closing Summary

By designing a clear separation of concerns, we ensure that the client remains lightweight and secure while still supporting complex navigation flows. The backend handles server-side logic, the client performs local validation, and deep links are supported through URL handling.

#### Common Mistakes/Follow-Ups
Overcomplicating Client-Side Logic: Ensure that the client only performs necessary validations to avoid security risks.
Exposing Backend Internals in URLs: Deep linking should not reveal backend state or implementation details.
Handling Edge Cases: Consider scenarios where user input fails validation and how the system responds.

**Code:**

```

class NavigationService: def init(self): self.navigation_states = {}

CODE
Copy
def add_navigation_state(self, state_id, next_screen):
    self.navigation_states[state_id] = next_screen

async def get_next_screen(self, current_state):
    return self.navigation_states.get(current_state)

navigation_service = NavigationService()

CODE
Copy

```javascript
class Client {
    constructor(navigationService) {
        this.navigationService = navigationService;
        this.validator = new Validator();
    }

    async handleNavigationRequest(stateId, userInput) {
        const nextScreen = await this.navigationService.get_next_screen(stateId);
        if (nextScreen && this.validator.validate(userInput)) {
            // Perform client-side rendering or redirection
        } else {
            console.error("Validation failed or no valid state found");
        }
    }
}

class Validator {
    validate(input) {
        return true; // Placeholder
    }
}
```

**Example input / output:**

```text
Input:
- Current State ID: 'screen1'
- User Input: { username: 'testuser', password: 'securepassword' }

Output:
- If validation passes, client navigates to Home Screen.
- If validation fails or no valid state found, an error message is logged.

Request:
GET /navigation/screen1

Response:
{
  "next_screen": {
    "id": "screen2",
    "name": "Home Screen"
  }
}
```

**Data:**

```json
{
  "id": "screen1",
  "name": "Login Screen",
  "next": {
    "id": "screen2",
    "name": "Home Screen"
  }
}
```

**Flow:**

1. Yes
2. No
3. Client Receives Request
4. Navigates to Server
5. Valid Input?
6. Navigates to Next Screen
7. Display Error and Stay onCurrent Screen

