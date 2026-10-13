---
type: "practice"
difficulty: "Deep"
---

# A bar-raiser asks how you'd model 'a mutation that spans client optimistic state, an API call, and a cache update' as one cohesive fullstack abstraction so a component author writes one thing and gets optimistic apply, rollback, and cache reconciliation for free; what's the class contract?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Two-level Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model 'a mutation that spans client optimistic state, an API call, and a cache update' as one cohesive fullstack abstraction, we need to design a system where a component author can write a single mutation function and have the system handle the optimistic UI updates, API calls, and cache reconciliation for them. The key idea is to encapsulate this complex process into a high-level class contract that abstracts away the underlying mechanics.

First, let's triage the question: the interviewer is probing how you would design a system that can manage optimistic UI updates, ensure consistent state across different layers (client, server), and handle cache coherence. The core concept involves leveraging modern JavaScript frameworks' capabilities for reactive programming and transactional handling of mutations.

#### Assumptions
We are using a fullstack framework like React with Redux or a similar state management solution.
The application has a RESTful API layer that can be called via HTTP requests.
The cache is managed through a caching layer, possibly Redis or another in-memory store.
Step-by-Step Solution
1. Define the Mutation Class

The class contract should encapsulate the mutation logic and ensure that it handles optimistic updates, API calls, and cache reconciliation seamlessly. Let’s define this class:

JAVASCRIPT
class Mutation {

#### cons
tructor(apiEndpoint) {
        this.apiEndpoint = apiEndpoint;
        // Additional properties like optimisticState, lastCommitTime, etc.
    }

    async performMutation(inputData) {
        try {
            // Step 1: Apply Optimistic State
            await this.optimisticallyApplyState(inputData);

            // Step 2: Make API Call

#### cons
t apiResponse = await this.makeApiCall(this.apiEndpoint, inputData);

            // Step 3: Reconcile Cache and Persist Data
            await this.reconcileCacheAndPersist(apiResponse);
        } catch (error) {
            // Handle rollback in case of failure
            this.rollbackOptimisticState();
            throw error;
        }
    }

    async optimisticallyApplyState(inputData) {
        // Apply optimistic state changes locally

#### cons
ole.log(`Applying optimistic state: ${JSON.stringify(inputData)}`);
        // Example implementation:
        // setState(state => ({ ...state, [inputKey]: inputData }))
    }

    async makeApiCall(endpoint, data) {

#### cons
t response = await fetch(endpoint, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data),
        });
        return await response.json();
    }

    async reconcileCacheAndPersist(responseData) {
        // Reconcile cache and persist data

#### cons
ole.log(`Reconciling cache with API response: ${JSON.stringify(responseData)}`);
        // Example implementation:
        // this.cacheService.set(this.apiEndpoint, responseData)
        // this.storageService.persist(this.apiEndpoint, responseData)
    }

    rollbackOptimisticState() {
        // Revert local state changes

#### cons
ole.log('Rolling back optimistic state');
        // Example implementation:
        // setState(state => ({ ...state, [inputKey]: previousValue }))
    }
}
2. Use the Mutation Class in Components

A component author can now use this class to perform mutations:

JAVASCRIPT
import { useState } from 'react';
import Mutation from './Mutation';

#### cons
t MyComponent = () => {

#### cons
t [data, setData] = useState({ key: '' });

#### cons
t mutation = new Mutation('/api/endpoint');

#### cons
t handleInputChange = (event) => {
        setData({ ...data, [event.target.name]: event.target.value });
    };

#### cons
t handleSubmit = async (event) => {
        event.preventDefault();
        try {
            await mutation.performMutation(data);

#### cons
ole.log('Mutation successful!');
        } catch (error) {

#### cons
ole.error('Mutation failed:', error);
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <input type="text" name="key" value={data.key} onChange={handleInputChange} />
            <button type="submit">Submit</button>
        </form>
    );
};

export default MyComponent;

#### Closing Summary

This solution encapsulates the mutation process into a high-level Mutation class, handling optimistic UI updates, API calls, and cache reconciliation. By using this abstraction, component authors can write simpler mutations that automatically benefit from these features without managing the underlying complexity.

#### Common Mistakes/Follow-Ups
Not Handling Errors Properly: Ensure proper error handling in both the mutation logic and rollback mechanism.
Inconsistent State Management: Make sure to handle state changes consistently across all layers (client, server, cache).

#### Performance Considerations
Cache invalidation and reconciliation should be optimized to avoid unnecessary API calls or cache thrashing.

**Comparison:**

```
Step	Action	Description
1	Apply Optimistic State	Modify local state immediately to reflect the new data.
2	Make API Call	Send a POST request to the API endpoint with the latest data.
3	Reconcile Cache and Persist	Update cache and persist the data if the API call succeeds or handle errors.
```

**Code:**

```javascript
class Mutation {
    // ...
}

const MyComponent = () => {
    const [data, setData] = useState({ key: '' });
    const mutation = new Mutation('/api/endpoint');

    const handleInputChange = (event) => {
        setData({ ...data, [event.target.name]: event.target.value });
    };

    const handleSubmit = async (event) => {
        event.preventDefault();
        try {
            await mutation.performMutation(data);
            console.log('Mutation successful!');
        } catch (error) {
            console.error('Mutation failed:', error);
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <input type="text" name="key" value={data.key} onChange={handleInputChange} />
            <button type="submit">Submit</button>
        </form>
    );
};
```

**Example input / output:**

```

Input:

JAVASCRIPT
Copy
const data = { key: 'new value' };

Output (assuming optimistic state is applied and API call succeeds):

PLAINTEXT
Copy
Applying optimistic state: {"key":"new value"}
Reconciling cache with API response: {"id":123,"key":"new value"}
Mutation successful!
```

**Data:**

```json
{
  "apiEndpoint": "/api/endpoint",
  "optimisticState": {
    "isPending": false,
    "data": {}
  },
  "lastCommitTime": null
}
```

**Flow:**

1. Start Mutation
2. Apply Optimistic State
3. Make API Call
4. API Call Succeeded
5. Reconcile Cache and Persist
6. Rollback Optimistic State
7. Mutation Successful!
8. Mutation Failed!

