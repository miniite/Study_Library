---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks how you'd model 'a component's derived state' so a value computed from props/state isn't stored redundantly (causing stale-state bugs) yet isn't recomputed wastefully; how do you set the memoization/selector boundary and where does it belong architecturally?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotent Consumer Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model a component's derived state without causing redundant storage or wasteful recomputation, we need to carefully define the boundaries of memoization and selectors. The core idea is to ensure that the derived state is computed only when necessary and stored efficiently. This approach requires understanding how the component's props and state interact and where the logic should reside in our application architecture.

Firstly, let’s clarify what "a component's derived state" means: it’s a value that depends on other state or props but isn’t directly stored as part of those values. For example, if you have an input field that calculates the length of its text content (derived state), storing this derived state could lead to stale-state issues when the text changes.

#### Triaging the Question

The interviewer is probing your understanding of memoization, selector functions, and how to optimize component re-renders in a React application. They are also interested in whether you can apply these concepts effectively to maintain clean and performant code.

Key Assumptions
We're working within a React application.
The derived state is computed based on props or state changes.
We want to avoid redundant computations and stale-state issues.

#### First-Principles Solution
1. Understanding Memoization vs Selector Functions

Memoization typically involves caching the result of a function so that it can be reused if the same input arguments are provided again, thereby avoiding recomputation. In contrast, selectors are functions used to derive state from store data in a Redux-like setup.

For our specific problem, we need to create memoized selector functions that compute derived states only when necessary and cache these results appropriately.

2. Defining Derived State Logic

We define a function that computes the derived state based on props or state. For example:

JAVASCRIPT
function computeDerivedState(text) {
  return text.length;
}
3. Memoizing the Derived State Computation

We use React's useMemo hook to memoize the computation of our derived state, ensuring it only recalculates when its dependencies change.

JAVASCRIPT
import { useMemo } from 'react';

function MyComponent({ text }) {

#### cons
t derivedState = useMemo(() => computeDerivedState(text), [text]);

  return <div>{derivedState}</div>;
}
4. Managing State and Props Efficiently

Ensure that the component’s state and props are managed efficiently to avoid unnecessary recomputations:

Use useEffect for side effects, like fetching data.
Utilize useCallback to memoize callback functions if they depend on other values.
Architectural Considerations

The derived state logic should be encapsulated within the component where it’s used. This keeps your components focused and reduces coupling with other parts of your application.

Code Example
JAVASCRIPT
// MyComponent.js
import { useMemo } from 'react';

function computeDerivedState(text) {
  return text.length;
}

function MyComponent({ text }) {

#### cons
t derivedState = useMemo(() => computeDerivedState(text), [text]);

  return <div>{derivedState}</div>;
}

#### Closing Summary
and Common Mistakes

By encapsulating the logic within the component, we ensure that:

Derived state is computed only when necessary.
Components are efficient and responsive.

#### Common mistakes
include:

Storing derived state: Avoid storing derived state in state or local variables unless absolutely necessary.
Redundant computations: Ensure that derived states are memoized to avoid unnecessary recomputations.
Overcomplicating logic: Keep the logic simple and focused on its purpose.

#### Conclusion

Effective modeling of a component's derived state involves using tools like useMemo for memoization, ensuring efficient prop/state management, and keeping the logic encapsulated within the relevant components. This approach helps in maintaining clean, performant, and scalable code.

**Code:**

```javascript
import { useMemo } from 'react';

function computeDerivedState(text) {
  return text.length;
}

function MyComponent({ text }) {
  const derivedState = useMemo(() => computeDerivedState(text), [text]);

  return <div>{derivedState}</div>;
}
```

