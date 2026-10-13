---
type: "practice"
difficulty: "Deep"
---

# Design an architecture where a mobile app must work across a spectrum from 5G to 2G, degrading gracefully: full experience, image-lite, then text-only. How does the client negotiate this with the backend, and where do you cache to survive intermittent connectivity?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Client Upgrades](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an architecture that supports a mobile app working across different network qualities—from 5G to 2G—while degrading gracefully (full experience → image-lite → text-only), we need to focus on several key components and strategies. The core idea is to provide a robust, flexible system capable of adapting to varying connection conditions while ensuring the best possible user experience at each level.

At its heart, this solution involves client-side logic that dynamically adjusts content based on network quality, a backend that serves progressively more or less rich content depending on the request, and strategic caching mechanisms. The client will negotiate with the backend using HTTP headers or custom JSON payloads to determine the appropriate content tier for each request.

#### Triaging the Question

The question is essentially asking how to design an application architecture that can handle different network conditions while providing a seamless user experience by adjusting the content complexity (full, image-lite, text-only) as the connection degrades. The interviewer is probing your understanding of client-server communication protocols, caching strategies, and performance optimization techniques.

#### First Principles Solution
Client-Side Adaptation: The mobile app must be able to detect network conditions in real-time and adjust its content requests accordingly.
Backend Adaptation: The backend should serve different levels of content based on the detected network quality.
Caching Strategy: Implement caching mechanisms to ensure that the client can continue functioning even when intermittent connectivity issues arise.
Detailed Solution

Network Detection: Use a combination of JavaScript and native platform APIs (e.g., navigator.onLine for web apps, Network Information API) to detect network status changes.

Client-Side Logic:

Full Experience: For optimal 5G or LTE networks, the app requests full content with rich media and real-time updates.
Image-Lite Mode: As the connection degrades (3G or 4G), the app switches to a mode that fetches fewer images but retains text-based content for faster load times.
Text-Only Mode: In 2G or lower networks, only minimal text is fetched and displayed.

Backend Design:

Create multiple APIs endpoints based on content complexity (full, image-lite, text-only).
Use HTTP headers (Accept, User-Agent) to determine the app's capabilities.
Implement a custom JSON payload in the response that includes the appropriate content level for each request.

Caching:

Utilize browser caching and local storage to store frequently accessed data and images.
Implement server-side caching (e.g., CDN, Varnish) to reduce load times and ensure data availability even during intermittent connectivity issues.

#### Closing Summary

By designing the system with a layered approach that adapts content based on network quality, we can provide a seamless user experience across different network conditions. The client uses real-time network detection and custom backend logic to serve appropriate content levels, while caching strategies help maintain functionality during connectivity disruptions.

#### Common Mistakes
and Follow-Ups
Mistake: Overcomplicating the server-side solution by creating too many complex APIs or endpoints.

#### Follow-Up
What cache invalidation strategy will you use for content that is frequently updated?
Mistake: Ignoring edge cases, such as hybrid networks or varying latency between devices and servers.

#### Follow-Up
How do you handle network fluctuation scenarios where the connection changes rapidly?

**Comparison:**

```
Network Quality	Content Level	Description
5G - LTE	full	Rich media, real-time updates
3G - 4G	image-lite	Minimal images, text-based content
2G or lower	text-only	Only text available
```

**Code:**

```javascript
// Example of client-side logic for detecting online status
navigator.onLine ? console.log("Online") : console.log("Offline");

// Example of backend negotiation using HTTP headers
const xhr = new XMLHttpRequest();
xhr.open('GET', '/content', true);
xhr.setRequestHeader('Accept', 'image-lite'); // or text-only, full

xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    console.log(xhr.responseText);
  }
};
xhr.send(null);
```

**Example input / output:**

```text
Input (Client Request):
GET /content

Response:
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Accept: image-lite

{
  "contentLevel": "image-lite",
  "data": [
    {"id": 1, "title": "Article Title", "content": "<p>Text-based content with minimal images</p>", "images": ["thumbnail1.png"]}
  ]
}
```

**Data:**

```json
{
  "contentLevel": "full",
  "data": [
    {"id": 1, "title": "Article Title", "content": "<p>Full content with rich media</p>", "images": ["image1.png", "image2.jpg"]}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Client Detects Online Status
6. Is Network Good?
7. Request Full Content
8. Is Network 3G/4G?
9. Request Image-Lite Content
10. Request Text-Only Content
11. Cache and Display Content
12. Cached or Fetch from Serverthen Cache, Display Image-Lite
13. Cached or Fetch from Serverthen Cache, Display Text-Only
14. Notify Client of Success/Failure

