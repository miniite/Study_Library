---
type: "practice"
difficulty: "Deep"
---

# Design an EdTech live-class platform delivering interactive classes to 100K concurrent students with video quizzes and doubt-solving — how do you architect the low-latency AV interactivity and recording so a class survives a regional network blip?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fan-out Trade-offs](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an EdTech live-class platform that can handle 100K concurrent students with interactive video quizzes and doubt-solving, we need to focus on low-latency audio-video (AV) interactivity and robust recording mechanisms. The architecture must ensure that the class continues seamlessly even if there's a regional network blip. Here’s how we can achieve this:

TL;DR

We'll design an architecture using a combination of edge locations, content delivery networks (CDNs), real-time communication protocols, and resilient storage solutions to ensure low-latency interactivity and robust recording.

#### Triaging the Question

The key elements are:

Low-Latency AV Interactivity: Ensuring minimal delay in audio-video streams.
Robust Recording Mechanism: Capturing video recordings that can withstand network disruptions.
Handling Regional Network Blips: Ensuring the class continues without significant disruption, even if regional networks fail.

#### Assumptions
The platform will have multiple edge locations to reduce latency.
We will use a CDN for efficient content delivery and caching.
Real-time communication (RTC) protocols like WebRTC or RTMP will be used for low-latency streaming.
Database systems will support high write speeds to manage student interactions.

#### First-Principles Solution
Edge Locations: Deploy edge nodes in various regions to minimize latency between the server and the students' devices.
CDN Integration: Use a CDN to cache content closer to the users, reducing network hops and improving performance.
RTC Protocols: Implement WebRTC or RTMP for real-time video streaming to ensure minimal latency.
Recording Mechanisms: Use segmented recording with high-speed storage and distributed file systems (like Amazon S3) for robustness.
Resilience Strategies:
Redundant Edge Nodes: Ensure that multiple edge nodes can take over in case of a failure.
Failover Mechanism: Implement a failover mechanism to switch seamlessly to an alternative CDN or edge node.
Detailed Design
1. Edge Locations and CDN

We will deploy edge locations globally, covering major regions where the platform is popular. Each edge location will serve as a cache for static content and will handle real-time traffic. The CDN will be integrated to ensure that frequently accessed content is cached at these edge nodes.

Global Edge Nodes
CDN Cache
User Devices
2. Real-Time Communication (RTC) Protocols

For live classes, we will use WebRTC or RTMP to ensure minimal latency and high-quality video streaming. These protocols are designed for real-time communication over the internet and can handle multiple users efficiently.

WebRTC/RTMP
Real-Time Video Streaming
3. Recording Mechanisms

We will use a segmented recording approach where each segment of video is stored in high-speed storage solutions like Amazon S3 or a distributed file system (DFS). This ensures that even if one node fails, the recording can still be recovered.

Segmented Video Recording
High-Speed Storage
4. Resilience Strategies

To handle regional network blips, we will implement a redundant edge node system and a failover mechanism. If an edge node fails, the traffic can be redirected to another available edge node or the CDN.

Redundant Edge Nodes
Failover Mechanism

#### Closing Summary
+ Common Mistakes/Follow-Ups

The architecture described above ensures low-latency AV interactivity and robust recording mechanisms. Key points include leveraging edge locations, CDNs, real-time communication protocols, and resilient storage solutions. Common mistakes might be overlooking the importance of redundancy or failing to implement a failover mechanism.

For follow-ups, you may need to consider issues like security, user authentication, and scalability under peak load conditions.

**Comparison:**

| Edge Node | Region       | Nodes   |
|-----------|-------------|---------|
| US-East-1  | United States| nodeA, nodeB |
| Europe-West-2 | Europe     | nodeC, nodeD |

**Code:**

```javascript
// Example WebRTC configuration for live classes
const config = {
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' }
  ],
  iceTransportPolicy: 'all',
  rtcpMuxPolicy: 'require'
};

const pc = new RTCPeerConnection(config);
```

**Data:**

```yaml
# Example CDN configuration
edgeLocations:
  - region: US-East-1
    nodes: [nodeA, nodeB]
  - region: Europe-West-2
    nodes: [nodeC, nodeD]
```

**Flow:**

1. User Connects to Edge Node
2. Stream Sent via WebRTC
3. CDN Cache Hits
4. Content Delivered to User
5. CDN Cache Miss
6. Content Served Directly fromOrigin

