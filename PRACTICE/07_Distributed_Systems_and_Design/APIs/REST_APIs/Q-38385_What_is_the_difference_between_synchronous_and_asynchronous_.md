---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between synchronous and asynchronous REST API calls from the client's perspective?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ REST Principles](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/REST_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing REST API calls, we often distinguish between synchronous and asynchronous calls from the client’s perspective. In simple terms, a synchronous call blocks the client until it receives a response, while an asynchronous call does not.

#### Triaging the Question

The interviewer is likely probing your understanding of how these two types of API interactions behave in different environments. They are also testing whether you can explain the implications and practical differences for real-world applications.

#### Assumptions
We are focusing on client-side behavior, meaning the perspective from which a web or mobile application interacts with an API.
Both synchronous and asynchronous calls involve HTTP requests to REST APIs.
The discussion will touch on performance impacts, user experience, and error handling.
Step-by-Step Solution:
Synchronous Calls:
Blocking Behavior: When making a synchronous call, the client code waits for the response before proceeding with further operations.
User Experience: This can lead to poor user experience if the API is slow or unresponsive, as the UI will be frozen until the response is received.
Error Handling: Simpler error handling since you can catch errors immediately after receiving a response.
Asynchronous Calls:
Non-blocking Behavior: Asynchronous calls allow the client code to continue executing other tasks while waiting for the API’s response.
User Experience: Provides better user experience because the UI remains interactive, enhancing the overall feel of responsiveness.
Error Handling: More complex due to the need to handle errors in a callback or promise chain.

#### First-Principles Solution

Let's break down these concepts with an example using JavaScript and AJAX calls for web applications.

#### Example Scenario
Fetching User Data

Synchronous Call (Example)

JAVASCRIPT
function getUserDataSync(userId) {

#### cons
t xhr = new XMLHttpRequest();
    xhr.open('GET', `/api/users/${userId}`, false); // Notice the 'false' parameter, making it synchronous
    xhr.send();

    if (xhr.status === 200) {
        return JSON.parse(xhr.responseText);
    } else {
        throw new Error('Failed to fetch user data');
    }
}

try {

#### cons
t userData = getUserDataSync(123);

#### cons
ole.log(userData);
} catch (error) {

#### cons
ole.error(error);
}

Asynchronous Call (Example)

JAVASCRIPT
function getUserDataAsync(userId) {
    return new Promise((resolve, reject) => {

#### cons
t xhr = new XMLHttpRequest();
        xhr.open('GET', `/api/users/${userId}`, true); // Notice the 'true' parameter, making it asynchronous
        xhr.onload = function () {
            if (xhr.status === 200) {
                resolve(JSON.parse(xhr.responseText));
            } else {
                reject(new Error('Failed to fetch user data'));
            }
        };
        xhr.onerror = function () {
            reject(new Error('Network error'));
        };
        xhr.send();
    });
}

getUserDataAsync(123)
    .then((userData) => console.log(userData))
    .catch((error) => console.error(error));

#### Closing Summary

In summary, synchronous calls block the client’s execution flow until they receive a response, which can degrade user experience in slower API interactions. Asynchronous calls, while more complex to implement, offer better performance and responsiveness by not blocking other tasks.

#### Common Mistakes/Follow-ups
Forgetting to handle errors: Synchronous calls often have simpler error handling, but asynchronous methods must manage errors using callbacks or promises.
Neglecting the impact on UI responsiveness: Asynchronous calls are crucial for keeping the user interface responsive and improving overall application performance.
Misunderstanding when to use synchronous vs. asynchronous: As a rule of thumb, always opt for asynchronous unless you have specific reasons to block execution.

**Code:**

```javascript
// Synchronous call example
function getUserDataSync(userId) {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', `/api/users/${userId}`, false);
    xhr.send();

    if (xhr.status === 200) {
        return JSON.parse(xhr.responseText);
    } else {
        throw new Error('Failed to fetch user data');
    }
}

// Asynchronous call example
function getUserDataAsync(userId) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open('GET', `/api/users/${userId}`, true);
        xhr.onload = function () {
            if (xhr.status === 200) {
                resolve(JSON.parse(xhr.responseText));
            } else {
                reject(new Error('Failed to fetch user data'));
            }
        };
        xhr.onerror = function () {
            reject(new Error('Network error'));
        };
        xhr.send();
    });
}
```

