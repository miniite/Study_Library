---
type: "practice"
difficulty: "Deep"
---

# You measure your AI endpoint at 700ms but users report 4s. How do you set up client-perceived latency measurement for streaming responses, and what does time-to-first-token vs total time tell you about where to optimize?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Progressive Results](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Progressive_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively measure and understand the client-perceived latency in a streaming API response, we need to break down the problem into its core components and leverage both server-side timing metrics and client-side measurement techniques. The key is to distinguish between time-to-first-token (TTFB) and total request latency, as these two metrics provide insights into different stages of the process.

#### Triage

The question asks us to set up a mechanism for measuring client-perceived latency specifically for streaming responses. We need to understand that this involves both server-side timing and client-side measurements. The user reports 4 seconds, which is significantly higher than the measured 700ms on the server side. This suggests that there are additional delays between when the server starts sending data and when it completes the stream.

Key Concepts
Server-Side Metrics: These include time-to-first-token (TTFB), which measures how long it takes for the first byte of data to be sent from the server.
Client-Side Measurement: Capturing the time taken from initiating the request until the client receives the first token of data and when the stream is fully complete.

#### First-Principles Solution

To set up a comprehensive measurement system:

Server-Side Setup:

Introduce middleware or instrumentation to measure TTFB.
Log timestamps at key points: before sending the first byte, after sending each chunk of data, and upon completion.

Client-Side Implementation:

Use JavaScript in a web client to measure the time between making the request and receiving the first token of data.
Record the total time until the stream is fully complete.

Data Collection and Analysis:

Aggregate these metrics over multiple requests to get an average and identify patterns or outliers.
Correlate server-side TTFB with client-side measurements to pinpoint where delays are occurring.
Detailed Steps

Server-Side Instrumentation:

JAVASCRIPT
// Example using Express.js middleware for logging timestamps

#### cons
t express = require('express');

#### cons
t app = express();

app.use((req, res, next) => {
    req.startTimestamp = Date.now();
    next();
});

app.get('/stream', (req, res) => {
    res.setHeader('Content-Type', 'text/event-stream; charset=utf-8');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    // Simulate streaming data

#### cons
t interval = setInterval(() => {
        res.write(`data: ${new Date().toISOString()}\n\n`);
        if (Date.now() - req.startTimestamp > 1000) { // Stop after 1 second for example
            clearInterval(interval);
            res.end();
        }
    }, 250);

    return;
});

app.listen(3000, () => {

#### cons
ole.log('Server running on port 3000');
});

Client-Side Measurement:

JAVASCRIPT
async function measureStreamingLatency() {

#### cons
t start = performance.now();

#### cons
t response = await fetch('/stream');

#### cons
t reader = response.body.getReader();

    try {
        let firstTokenRead;
        while (true) {

#### cons
t { done, value } = await reader.read();
            if (done) break;
            // Store the time of receiving the first token
            firstTokenRead = performance.now() - start;

#### cons
ole.log(`First token received at: ${firstTokenRead.toFixed(2)} ms`);
        }
    } finally {

#### cons
t totalTime = performance.now() - start;

#### cons
ole.log(`Total stream latency: ${totalTime.toFixed(2)} ms`);
    }
}

measureStreamingLatency();

#### Closing Summary

By setting up both server-side and client-side measurements, we can accurately capture the time-to-first-token (TTFB) as well as the total streaming latency. This dual approach helps identify where delays occur in the pipeline: whether they are on the server end or due to network conditions, browser limitations, etc.

#### Common Mistakes/Follow-Ups
Incorrect Timing: Ensure that TTFB is correctly calculated from the first byte sent by the server.
Network Delays: Consider external factors like internet latency and how they affect the client's perception of latency.
Browser Capabilities: Streaming responses can be affected by browser limitations, such as chunked encoding handling.

Understanding these differences helps in pinpointing areas for optimization, whether it’s improving backend performance or optimizing network conditions.

**Code:**

```

// Server-side middleware for TTFB logging app.use((req, res, next) => { req.startTimestamp = Date.now(); next(); });

CODE
Copy

```javascript
// Client-side measurement script
async function measureStreamingLatency() {
    const start = performance.now();
    const response = await fetch('/stream');
    const reader = response.body.getReader();

    try {
        let firstTokenRead;
        while (true) {
            const { done, value } = await reader.read();
            if (done) break;
            // Store the time of receiving the first token
            firstTokenRead = performance.now() - start;
            console.log(`First token received at: ${firstTokenRead.toFixed(2)} ms`);
        }
    } finally {
        const totalTime = performance.now() - start;
        console.log(`Total stream latency: ${totalTime.toFixed(2)} ms`);
    }
}

measureStreamingLatency();
```

**Flow:**

1. Initiate Request
2. Server-Side Middleware
3. Start Timestamp
4. Send First Byte
5. Client-Side Script
6. Record Start Time
7. Read Data Tokens
8. First Token Received?
9. Yes
10. Log TTFB
11. Total Latency Recorded?
12. Yes
13. End Stream
14. No
15. Log Total Time
16. Complete Measurement

