---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle and display loading, empty, and error states in the UI when consuming a REST endpoint?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ HTTP Status Codes](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Status_Codes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively handle and display loading, empty, and error states in the UI when consuming a REST endpoint, we need to ensure that our application provides a seamless user experience by managing these different scenarios appropriately. This involves making sure that the API calls are properly managed, and the frontend responds gracefully to various server responses.

#### Triage

The question is essentially asking how to handle different server responses (loading state when the request is in progress, empty state if no data is returned, error state if there's an issue with the API) and display them appropriately on the UI. The interviewer might be probing your understanding of asynchronous programming, handling states in React or Vue, and best practices for user feedback.

#### First-Principles Solution
API Request Management: Ensure that the REST API call is made asynchronously.
State Management: Track the state (loading, success, error) on the frontend.
Conditional Rendering: Use conditional rendering in the UI to display different components based on the current state.
Detailed Steps

Asynchronous API Call:

Make an asynchronous call using fetch, axios, or any other HTTP client library.
JAVASCRIPT
async function fetchData() {
  try {

#### cons
t response = await fetch('/api/endpoint');
    if (!response.ok) throw new Error('Network response was not ok');
    return await response.json();
  } catch (error) {

#### cons
ole.error('Failed to fetch data:', error);
    return null;
  }
}

State Management:

Use a state management library like Redux, Context API in React, or Vuex in Vue.
JAVASCRIPT
import { useState } from 'react';

function DataComponent() {

#### cons
t [data, setData] = useState(null);

#### cons
t [loading, setLoading] = useState(true);

#### cons
t [error, setError] = useState(false);

  useEffect(() => {
    fetchData().then(response => {
      if (response) {
        setData(response);
      } else {
        setError(true);
      }
      setLoading(false);
    });
  }, []);

  return (
    <div>
      {loading && <p>Loading...</p>}
      {!loading && !error && data && (
        <ul>
          {data.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      )}
      {loading && <p>No Data Found</p>}
      {error && <p>Error fetching data</p>}
    </div>
  );
}

Conditional Rendering:

Use conditional rendering to show different UI components based on the state.
JAVASCRIPT
return (
  <>
    {loading && <p>Loading...</p>}
    {!loading && !error && data && (
      <ul>
        {data.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    )}
    {loading && <p>No Data Found</p>}
    {error && <p>Error fetching data</p>}
  </>
);

#### Closing Summary

In summary, handling loading, empty, and error states in a REST API response requires proper state management and conditional rendering. By tracking the state (loading, success, error) and updating the UI accordingly, we can provide a smooth user experience.

#### Common Mistakes/Follow-Ups
Common Mistake: Not properly managing asynchronous calls, leading to race conditions or incorrect handling of states.

#### Follow-Up
Question: How would you handle pagination in this scenario? What about implementing retries for temporary errors?

**Code:**

```javascript
// Asynchronous API Call
async function fetchData() {
  try {
    const response = await fetch('/api/endpoint');
    if (!response.ok) throw new Error('Network response was not ok');
    return await response.json();
  } catch (error) {
    console.error('Failed to fetch data:', error);
    return null;
  }
}

// State Management in React
import { useState, useEffect } from 'react';

function DataComponent() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);

  useEffect(() => {
    fetchData().then(response => {
      if (response) {
        setData(response);
      } else {
        setError(true);
      }
      setLoading(false);
    });
  }, []);

  return (
    <div>
      {loading && <p>Loading...</p>}
      {!loading && !error && data && (
        <ul>
          {data.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      )}
      {loading && <p>No Data Found</p>}
      {error && <p>Error fetching data</p>}
    </div>
  );
}
```

