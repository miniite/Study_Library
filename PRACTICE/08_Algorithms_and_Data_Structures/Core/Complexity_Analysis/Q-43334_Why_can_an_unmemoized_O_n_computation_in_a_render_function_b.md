---
type: "practice"
difficulty: "Intermediate"
---

# Why can an unmemoized O(n) computation in a render function become effectively O(n * renders), and how does useMemo help?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Streaming vs In-memory Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Streaming_vs_In-memory_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you have a render loop that performs an (O(n)) operation without memoization, it can end up being executed multiple times per render due to the nature of React's reconciliation process. useMemo helps mitigate this by caching the result of an expensive computation and only recomputing when necessary.

TL;DR

Unmemoized (O(n)) computations in a render function can become effectively (O(n \times \text{renders})) because they are recalculated every time the component renders. useMemo helps by memoizing the result, ensuring that the computation is only performed if the dependencies change.

#### Triaging the Question

The question asks why an unmemoized (O(n)) operation in a render function can become effectively (O(n \times \text{renders})), and how useMemo addresses this issue. The key here is understanding React's reconciliation process and memoization techniques.

#### Assumptions
We are working with a fullstack SDE who has intermediate experience.
The question assumes knowledge of the React lifecycle and performance optimization techniques like memoization.

#### First-Principles Solution

In React, each render cycle involves re-evaluating the entire component tree to determine which parts need to be updated. If you have an (O(n)) operation inside a render function that depends on n pieces of data, it will execute every time the component re-renders.

For example, consider a simple scenario where you are rendering an array of items:

JSX
function MyComponent() {

#### cons
t [items, setItems] = useState([1, 2, 3, 4, 5]);

  // Unmemoized O(n) operation in render function
  return (
    <div>
      {items.map((item) => (
        <span key={item}>{item}</span>
      ))}
    </div>
  );
}

Here, the map function is an (O(n)) operation. Each time MyComponent re-renders (e.g., due to a state update), this map function will execute over all elements in the array.

Now imagine a more complex scenario where you are applying some transformation or computation on each item:

JSX
function MyComponent() {

#### cons
t [items, setItems] = useState([1, 2, 3, 4, 5]);

#### cons
t transformedItems = items.map((item) => item * 2);

  return (
    <div>
      {transformedItems.map((item) => (
        <span key={item}>{item}</span>
      ))}
    </div>
  );
}

In this case, the map function is still (O(n)). However, if you are performing a costly operation (like complex computations or fetching data), it could be better to memoize the result.

How useMemo Helps

React’s useMemo hook helps by caching the result of an expensive computation. It returns a value that only changes if one of its dependencies change:

JSX
import { useMemo } from 'react';

function MyComponent() {

#### cons
t [items, setItems] = useState([1, 2, 3, 4, 5]);

  // Memoized version using useMemo

#### cons
t transformedItems = useMemo(() => items.map((item) => item * 2), [items]);

  return (
    <div>
      {transformedItems.map((item) => (
        <span key={item}>{item}</span>
      ))}
    </div>
  );
}

In this example, useMemo ensures that the transformation only happens when items changes. This prevents unnecessary computations and can significantly improve performance.

#### Closing Summary

Unmemoized (O(n)) operations in a render function are recalculated every time the component re-renders, leading to an effective complexity of (O(n \times \text{renders})). useMemo addresses this by memoizing results based on dependencies, reducing redundant computations and improving performance.

#### Common Mistakes/Follow-Ups
Not Understanding Dependencies: Forgetting that useMemo depends on the presence of the dependency array.
Overusing Memoization: Applying memoization to simple operations where it doesn’t provide much benefit.
Incorrect Usage: Not considering edge cases, such as when dependencies change unexpectedly.

Would you like me to elaborate on any part or add more examples?

**Code:**

```
JSX
Copy
// Example of unmemoized render function
function MyComponent() {
  const [items, setItems] = useState([1, 2, 3, 4, 5]);

  return (
    <div>
      {items.map((item) => (
        <span key={item}>{item * 2}</span>
      ))}
    </div>
  );
}

// Example of memoized render function using useMemo
import { useMemo } from 'react';

function MyComponent() {
  const [items, setItems] = useState([1, 2, 3, 4, 5]);

  const transformedItems = useMemo(() => items.map((item) => item * 2), [items]);

  return (
    <div>
      {transformedItems.map((item) => (
        <span key={item}>{item}</span>
      ))}
    </div>
  );
}
```

