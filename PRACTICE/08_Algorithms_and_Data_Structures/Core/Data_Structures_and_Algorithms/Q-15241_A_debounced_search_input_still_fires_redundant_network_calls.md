---
type: "practice"
difficulty: "Intermediate"
---

# A debounced search input still fires redundant network calls under fast typing; explain the queue or timer structure you'd use and how you cancel in-flight stale requests correctly.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once Dedup Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of redundant network calls in a debounced search input, we need to ensure that only the latest user interaction triggers a request while any ongoing or previous requests are cancelled. This can be effectively achieved using a queue with a timer or by implementing a timeout mechanism. Let's break down the solution step by step.

First, let’s clarify what the question is asking: We need to implement a debounce mechanism in a search input scenario to avoid making multiple network calls when the user types rapidly and then cancel any stale requests that are still pending.

#### Triaging the Question

The key aspects being probed include:

Understanding of Debounce Mechanism: The concept of debouncing, which ensures that an action does not trigger multiple times in quick succession.
Handling of In-flight Requests: Ensuring that only one request is processed at a time and any previous requests are cancelled.
Efficiency Considerations: Balancing between reducing network load while still providing responsive user experience.

#### Assumptions
The search input triggers a network call to fetch data from an API.
We need to ensure that the latest user interaction always wins, and any stale requests should be canceled.
Network calls are synchronous for simplicity; in practice, they would typically be asynchronous.

#### First Principles Solution

Queue Mechanism:

Use a queue to manage incoming search queries.
When a new query is received, add it to the queue.
Before adding a new item, check if there’s an ongoing request and cancel it.
Once the queue is empty or a timeout occurs, send the latest query as a network request.

Timeout Mechanism:

Use a single timer that resets on every input event.
When the timer expires, make the network call with the latest input value.
Clear the timer before setting it again to ensure only one request is made per timeout period.

#### Implementation
Example

Here’s an implementation using JavaScript and the setTimeout mechanism:

JAVASCRIPT
let debounceTimeout;

function handleSearchInput(query) {
    // Cancel any ongoing requests
    clearTimeout(debounceTimeout);

    // Reset the timer with a new request
    debounceTimeout = setTimeout(() => {
        makeNetworkRequest(query);
    }, 300); // 300ms debounce time
}

function makeNetworkRequest(query) {
    // Make API call to fetch data based on query

#### cons
ole.log(`Fetching data for: ${query}`);
}

#### Closing Summary
and Common Mistakes

In this solution, we’ve ensured that only the latest search input triggers a network request while any previous requests are cancelled. This approach balances between reducing redundant calls and maintaining responsiveness.

#### Common Mistakes
to Avoid:

Not cancelling ongoing requests when new inputs are received.
Setting up multiple timers independently without a central mechanism.
Failing to debounce the entire process, leading to excessive API calls.

**Code:**

```javascript
let debounceTimeout;

function handleSearchInput(query) {
    // Cancel any ongoing requests
    clearTimeout(debounceTimeout);

    // Reset the timer with a new request
    debounceTimeout = setTimeout(() => {
        makeNetworkRequest(query);
    }, 300); // 300ms debounce time
}

function makeNetworkRequest(query) {
    // Make API call to fetch data based on query
    console.log(`Fetching data for: ${query}`);
}
```

**Example input / output:**

```text
// Example usage:
handleSearchInput("search term 1");
handleSearchInput("search term 2"); // Cancels the first request and schedules a new one

setTimeout(() => {
    handleSearchInput("search term 3"); // This will also cancel any ongoing requests
}, 50);
```

**Flow:**

1. No
2. Yes
3. B
4. Cancels all in-flight requests
5. Sets new debounce timer
6. Timer Expires?
7. Make network request withlatest query
8. Wait for next input or timeout

