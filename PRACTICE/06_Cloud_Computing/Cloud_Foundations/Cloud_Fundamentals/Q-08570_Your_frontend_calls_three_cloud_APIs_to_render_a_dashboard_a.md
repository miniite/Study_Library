---
type: "practice"
difficulty: "Intermediate"
---

# Your frontend calls three cloud APIs to render a dashboard and one is slow. How would you keep the page usable and instrument which upstream is the bottleneck from the client side?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency Investigation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Tail_Latency_Investigation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a single API call in a frontend application that fetches data from three cloud APIs is slow, we need to ensure the page remains responsive and usable while also identifying which upstream service is causing the delay. This involves implementing caching strategies on the client side, optimizing asynchronous calls, and logging performance metrics.

#### Triaging the Question

The core of this question revolves around:

Maintaining User Experience: Ensuring that the dashboard remains functional despite one slow API.
Identifying Bottlenecks: Determining which cloud API is the bottleneck without a direct server-side inspection.

#### Assumptions
and Approach
The frontend application makes three concurrent requests to different APIs.
One of these requests is significantly slower, leading to an unresponsive UI for that part of the dashboard.
We need to implement a solution on the client side that handles this issue gracefully.

#### First Principles Solution
1. Asynchronous Fetch Requests

Firstly, we should ensure all API calls are made asynchronously using Promise or async/await. This ensures that even if one request takes longer than expected, it doesn't block the execution of other requests and the overall UI remains responsive.

JAVASCRIPT
async function fetchDashData() {

#### cons
t [api1Response, api2Response, api3Response] = await Promise.all([
    fetch('https://api.example.com/data1'),
    fetch('https://api.example.com/data2'),
    fetch('https://api.example.com/data3')
  ]);

#### cons
t data1 = await api1Response.json();

#### cons
t data2 = await api2Response.json();

#### cons
t data3 = await api3Response.json();

  // Render the dashboard
}
2. Loading Indicators and Progress Bars

To keep the UI usable, we can display loading indicators or progress bars for each section of the dashboard that is dependent on API calls. This informs the user that some parts are still being loaded.

HTML
<div id="dashboard">
  <div id="data1" class="loading-indicator"></div>
  <div id="data2" class="loading-indicator"></div>
  <div id="data3" class="loading-indicator"></div>
</div>
3. Caching Responses

Implement caching on the client side to store responses from APIs that are less frequently updated or have a consistent pattern of requests. This can significantly reduce latency.

JAVASCRIPT

#### cons
t cache = new Map();

function fetchWithCache(url) {
  if (cache.has(url)) {
    return Promise.resolve(cache.get(url));
  }

#### cons
t response = fetch(url);
  response.then(data => cache.set(url, data)).catch(error => console.error('Error fetching', url, error));

  return response;
}
4. Error Handling and Retries

Add robust error handling to manage unexpected delays or failures in API calls. Implement retries for critical requests with exponential backoff.

JAVASCRIPT
async function fetchWithRetries(url) {

#### cons
t maxRetries = 3;
  let retryCount = 0;

  while (retryCount < maxRetries) {
    try {

#### cons
t response = await fetch(url);
      if (!response.ok) throw new Error('Network response was not ok');
      return response;
    } catch (error) {
      retryCount++;

#### cons
ole.log(`Retry attempt ${retryCount} for ${url}`);
      // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, 2 ** retryCount * 1000));
    }
  }

  throw new Error('Failed to fetch after max retries');
}
5. Logging and Monitoring

Log the duration of each API call to identify the bottleneck. Use client-side tools like console.log or more sophisticated logging frameworks.

JAVASCRIPT
function logApiCall(url, startTime) {

#### cons
t endTime = Date.now();

#### cons
ole.log(`API ${url} took ${endTime - startTime} ms`);
}

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing these strategies, we ensure that the frontend application remains responsive even when one API call is slow. Additionally, logging helps in identifying which upstream service is causing delays.

#### Common Mistakes

Not handling errors properly, leading to a completely unresponsive UI.
Ignoring caching mechanisms, resulting in unnecessary network requests.
Failing to implement robust retries for critical data, leading to incomplete or inconsistent dashboard updates.

For follow-ups, the interviewer might ask about optimizing server-side performance, implementing more advanced client-side optimizations like Web Workers, or discussing how this issue scales with a growing number of concurrent users.

**Comparison:**

```
Request	Time Taken (ms)
Data1	200
Data2	500
Data3	1000
```

**Code:**

```

async function fetchDashData() { const [api1Response, api2Response, api3Response] = await Promise.all([ fetch('https://api.example.com/data1'), fetch('https://api.example.com/data2'), fetch('https://api.example.com/data3') ]);

const data1 = await api1Response.json(); const data2 = await api2Response.json(); const data3 = await api3Response.json();

// Render the dashboard }

CODE
Copy

```javascript
function fetchWithCache(url) {
  if (cache.has(url)) {
    return Promise.resolve(cache.get(url));
  }

  const response = fetch(url);
  response.then(data => cache.set(url, data)).catch(error => console.error('Error fetching', url, error));

  return response;
}
JAVASCRIPT
Copy
async function fetchWithRetries(url) {
  const maxRetries = 3;
  let retryCount = 0;

  while (retryCount < maxRetries) {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error('Network response was not ok');
      return response;
    } catch (error) {
      retryCount++;
      console.log(`Retry attempt ${retryCount} for ${url}`);
      // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, 2 ** retryCount * 1000));
    }
  }

  throw new Error('Failed to fetch after max retries');
}
JAVASCRIPT
Copy
function logApiCall(url, startTime) {
  const endTime = Date.now();
  console.log(`API ${url} took ${endTime - startTime} ms`);
}
```

**Example input / output:**

```text
// Example input and output for logging API call duration
const startTime = Date.now();
logApiCall('https://api.example.com/data3', startTime);
console.log(`API https://api.example.com/data3 took ${Date.now() - startTime} ms`);
```

**Data:**

```json
{
  "url": "https://api.example.com/data3",
  "startTime": 1640995200000,
  "endTime": 1640995205000
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is API slow?
5. Log API duration
6. Render dashboard section withdata
7. Identify bottleneck via logs
8. Optimize upstream service ifneeded

