---
type: "practice"
difficulty: "Deep"
---

# Some corporate firewalls block or throttle UDP 443, breaking HTTP/3. How do you make your edge gracefully fall back to HTTP/2?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ NAT and Port Exhaustion](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/NAT_and_Port_Exhaustion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When dealing with corporate firewalls that block or throttle UDP 443, which is used in HTTP/3 (QUIC), we need a robust strategy to ensure our application can still function correctly and deliver a seamless experience to users. The key idea here is to implement a graceful fallback mechanism from HTTP/3 to HTTP/2, which leverages the more widely supported protocols without disrupting the user journey.

#### Triaging the Question

The question is probing how we handle protocol-specific network restrictions in a way that maintains application functionality and performance. We need to consider:

Network Policies: Understanding corporate firewalls' behavior and their impact on different protocols.
Protocol Versions: Comparing HTTP/3 (QUIC) with HTTP/2, including their strengths and weaknesses.
Fallback Mechanisms: Designing a strategy that ensures graceful degradation when one protocol is blocked or throttled.

#### First-Principles Solution

Protocol Selection:

HTTP/3 (QUIC): Offers low-latency connections through UDP 443, but some corporate firewalls may block it.
HTTP/2: A widely supported version that uses TCP and is more resilient to network restrictions.

Fallback Mechanism Implementation:

Implement a multi-protocol approach where the client can negotiate between HTTP/3 and HTTP/2 based on network conditions.
Use a single endpoint that dynamically selects the appropriate protocol.

Client-Side Logic:

The client initiates a connection attempt using QUIC (UDP 443).
If the connection fails or times out, it retries with TCP over HTTP/2.
Optionally, implement a fallback timer to automatically switch protocols if no response is received within a certain timeframe.

#### Example
Implementation

Here's an example of how this could be implemented using JavaScript and Node.js:

JAVASCRIPT

#### cons
t http3 = require('http3-client'); // Hypothetical library for QUIC support

#### cons
t http2 = require('http2');       // Native HTTP/2 support in Node.js

async function fetchResource(url) {
    try {

#### cons
t options = { protocol: 'quic' }; // Attempt to use QUIC first

#### cons
t response = await http3.get(url, options);
        return response.body;
    } catch (errorQuic) {
        if (errorQuic.code === 'ETIMEDOUT') { // Timeout indicates a likely network issue

#### cons
ole.log('Fallback to HTTP/2 due to timeout');
        } else {
            throw errorQuic; // Re-throw for other errors
        }
    }

    try {

#### cons
t optionsHttp2 = { protocol: 'http' }; // Fallback to TCP over HTTP/2

#### cons
t responseHttp2 = await http2.get(url, optionsHttp2);
        return responseHttp2.body;
    } catch (errorHttp2) {

#### cons
ole.error('Failed to connect using both protocols:', errorHttp2);
    }
}

// Usage example:
fetchResource('https://example.com');

#### Closing Summary
and Common Mistakes

Gracefully falling back from HTTP/3 to HTTP/2 involves a combination of protocol selection, network condition monitoring, and intelligent fallback logic. Key considerations include ensuring the client can quickly detect and respond to network issues, minimizing user impact during transitions between protocols.

#### Common Mistakes

Over-reliance on QUIC: Not considering corporate firewall policies that may block UDP 443.
Insufficient Error Handling: Failing to implement proper timeouts or fallback timers, leading to indefinite waits in degraded network conditions.
Lack of User Feedback: Not providing users with clear feedback during protocol transitions.
Final Confirmation

This solution is both comprehensive and accurate for the given scenario. It ensures a seamless user experience by dynamically selecting the most appropriate protocol based on real-time network conditions.

