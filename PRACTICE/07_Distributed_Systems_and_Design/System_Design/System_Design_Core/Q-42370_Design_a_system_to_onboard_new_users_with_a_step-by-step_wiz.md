---
type: "practice"
difficulty: "Basic"
---

# Design a system to onboard new users with a step-by-step wizard that saves progress.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Chat History Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Chat_History_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

This design will outline a basic yet comprehensive approach for onboarding new users through a step-by-step wizard, ensuring user data is saved and accessible at every stage of the process. The key idea is to maintain state across sessions using session storage or cookies while utilizing a backend to persist data in a database.

#### Triaging the Question

The interviewer is probing your understanding of state management, user experience, and basic system design principles. Key assumptions include handling different types of users (e.g., guest vs logged-in), ensuring data integrity, and providing a smooth onboarding process without requiring too many page reloads or sessions.

From First Principles to Solution:

User Interface Design:

The wizard should be split into multiple steps, each with specific fields.
Each step should have a "Next" button that saves the current state before moving forward.
A "Back" button allows users to revisit previous steps.

State Management:

For guest users, we can use session storage or local storage (via cookies) to maintain user progress.
JAVASCRIPT

#### cons
t step = localStorage.getItem('userStep');
if (!step) {
  localStorage.setItem('userStep', 'step1'); // default start step
}
For logged-in users, the backend should handle state management via a session cookie or JWT (JSON Web Token).

Backend Design:

A RESTful API is necessary to save and retrieve user data.
Each API call will update the database with the current state of the wizard.

Database Schema:

SQL
CREATE TABLE UserOnboarding (
  id INT AUTO_INCREMENT PRIMARY KEY,
  userId VARCHAR(255) NOT NULL,
  stepNumber INT NOT NULL DEFAULT 1, -- current step number
  data TEXT, -- serialized form of user input
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

API Endpoints:

HTTP
POST /api/onboarding/start
# Initiates the onboarding process and sets initial state

PUT /api/onboarding/step/:stepNumber
# Updates the state for a specific step

Error Handling and User Experience:

Provide clear error messages if an API request fails.
Use loading indicators to inform users that data is being saved.

#### Closing Summary

This design ensures that user progress is maintained across sessions while providing a seamless onboarding experience. It leverages both frontend (session storage) and backend (APIs, database) technologies to achieve this goal efficiently.

#### Common Mistakes
and Follow-Ups:

Mistake: Forgetting to persist data on every step change.

#### Follow-Up
Always update the state in both session/local storage and the backend database.

Mistake: Not handling edge cases like form validation errors.

#### Follow-Up
Ensure that validation is performed both client-side and server-side.

Mistake: Ignoring user experience, e.g., not showing a progress indicator or providing clear navigation.

#### Follow-Up
Focus on UX/UI design to make the process intuitive for new users.

**Comparison:**

```
Step	Fields
1	Name, Email
2	Password, Confirm Password
```

**Code:**

```javascript
// Example of saving state in local storage and making an API call
function saveUserProgress(stepNumber) {
  const formData = document.getElementById('step' + stepNumber).serialize();
  localStorage.setItem('userStep', stepNumber);

  fetch('/api/onboarding/step/' + stepNumber, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(formData)
  }).then(response => response.json())
    .then(data => console.log('Success:', data))
    .catch((error) => console.error('Error:', error));
}
```

**Example input / output:**

```text
Input:
- User starts onboarding at step 1.
- Fills out name and email.

Output:
- State saved in local storage as userStep = '1'.
- API call made to /api/onboarding/step/1 with the form data.
- Onboarded user navigates to step 2.
```

**Data:**

```json
{
  "step1": {
    "name": "",
    "email": ""
  },
  "step2": {
    "password": "",
    "confirmPassword": ""
  }
}
```

**Flow:**

1. User Starts Onboarding
2. Step 1 - Name/Email
3. Save Step 1 Data (LocalStorage + API Call)
4. Step 2 - Password/ConfirmPassword
5. Save Step 2 Data (LocalStorage + API Call)
6. Completion Page / Next Steps

