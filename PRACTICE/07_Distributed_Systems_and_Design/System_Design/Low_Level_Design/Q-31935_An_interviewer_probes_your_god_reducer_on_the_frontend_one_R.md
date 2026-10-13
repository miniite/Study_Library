---
type: "practice"
difficulty: "Deep"
---

# An interviewer probes your 'god reducer' on the frontend: one Redux/store slice has grown to handle 40 action types and it's the merge-conflict hotspot; how do you decompose the state model into cohesive slices with clear ownership without breaking selectors across the app?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Design Pattern Governance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a monolithic Redux slice that has grown to 40 action types and is causing merge conflicts, it's crucial to decompose the state model into smaller, more manageable slices while preserving the integrity of selectors across the application. The core idea here is to refactor the existing state structure by identifying cohesive units that can operate independently yet coherently with other parts of the app.

#### Triage

The interviewer is probing how you manage complexity in a Redux application, ensuring maintainability and reducing merge conflicts. Key assumptions include:

You are familiar with the current state management pattern used (e.g., Redux).
The app relies on selectors for state retrieval.
There are concerns about code organization and refactoring.

#### First Principles Solution
Identify Cohesive Units: Begin by analyzing the 40 action types to categorize them into groups that logically belong together, such as user actions, UI state management, or data fetching.
Define Ownership: For each group of related actions, define clear ownership and responsibility for a slice of the state model.
Preserve Selectors Integrity: Ensure that existing selectors work seamlessly with the new state structure to avoid breaking changes.
Implement Reducers Separately: Write separate reducers for each slice, ensuring they handle their specific action types.
Update State Shapes: Adjust the overall state shape to accommodate the new slices without disrupting selectors.
Step-by-Step Solution

Analyze Action Types:

Review all 40 actions and categorize them into meaningful groups (e.g., user, ui, dataFetching).

Define Slices:

Create a slice for each group, e.g., userSlice, uiSlice, dataFetchingSlice.
Define clear interfaces for these slices to ensure they handle their respective actions and state.

Update Reducers:

For each action type, implement the corresponding reducer in its respective slice.
JAVASCRIPT
// Example: User Slice

#### cons
t userReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'USER_LOGIN':
      return { ...state, isAuthenticated: true };
    case 'USER_LOGOUT':
      return { ...state, isAuthenticated: false };
    default:
      return state;
  }
};

// Example: UI Slice

#### cons
t uiReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'TOGGLE_SIDEBAR':
      return { ...state, isSidebarOpen: !state.isSidebarOpen };
    default:
      return state;
  }
};

Update State Shape:

Modify the overall state shape to include these slices.
JAVASCRIPT

#### cons
t initialState = {
  user: { isAuthenticated: false },
  ui: { isSidebarOpen: false },
  dataFetching: { isLoading: false, error: null }
};

Refactor Selectors:

Ensure that existing selectors work with the new state structure.
JAVASCRIPT

#### cons
t selectIsUserAuthenticated = (state) => state.user.isAuthenticated;

#### cons
t selectSidebarStatus = (state) => state.ui.isSidebarOpen;

Test Thoroughly:

Write tests to cover each slice and ensure that selectors still work as expected.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By decomposing the state model, we enhance maintainability and reduce merge conflicts. Key mistakes include not defining clear interfaces for slices or neglecting to update existing selectors. Common follow-ups might involve discussing tools like immer for immutable updates in Redux or exploring other state management solutions if the complexity continues to grow.

Self-Check

Correctness + completeness confirmed.

**Comparison:**

| Action Type | Slice | Reducer | Selector |
|-------------|-------|---------|----------|
| USER_LOGIN  | user  | Yes     | Yes      |
| TOGGLE_SIDEBAR | ui   | Yes     | Yes      |
| FETCH_DATA  | dataFetching | Yes    | Yes      |

**Code:**

```javascript
// Example Reducer Implementation
const userReducer = (state = { isAuthenticated: false }, action) => {
  switch (action.type) {
    case 'USER_LOGIN':
      return { ...state, isAuthenticated: true };
    case 'USER_LOGOUT':
      return { ...state, isAuthenticated: false };
    default:
      return state;
  }
};

const uiReducer = (state = { isSidebarOpen: false }, action) => {
  switch (action.type) {
    case 'TOGGLE_SIDEBAR':
      return { ...state, isSidebarOpen: !state.isSidebarOpen };
    default:
      return state;
  }
};
```

**Example input / output:**

```text
- **Input**: Action `USER_LOGIN` dispatched.
- **Output**: State updated to `{ "user": { "isAuthenticated": true }, "ui": { "isSidebarOpen": false }, "dataFetching": { "isLoading": false, "error": null } }`.

- **Input**: Action `TOGGLE_SIDEBAR` dispatched.
- **Output**: State updated to `{ "user": { "isAuthenticated": false }, "ui": { "isSidebarOpen": true }, "dataFetching": { "isLoading": false, "error": null } }`.
```

**Data:**

```json
{
  "user": { "isAuthenticated": false },
  "ui": { "isSidebarOpen": false },
  "dataFetching": { "isLoading": false, "error": null }
}
```

**Flow:**

1. No
2. Yes
3. Action Dispatched
4. Is Action Type Known?
5. Update State Directly
6. Get Slice from Action Type
7. Run Reducer on Slice State
8. Return Updated State to Store
9. Use Selector for Selecting Datain Components

