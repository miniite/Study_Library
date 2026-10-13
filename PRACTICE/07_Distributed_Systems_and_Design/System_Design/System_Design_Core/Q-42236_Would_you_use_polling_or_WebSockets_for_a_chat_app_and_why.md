---
type: "practice"
difficulty: "Intermediate"
---

# Would you use polling or WebSockets for a chat app and why?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue vs Synchronous Calls](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Queue_vs_Synchronous_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: For a chat app, would you pick polling or WebSockets, and why? A good answer names the real options (short polling, long polling, SSE, WebSocket), reasons about them on latency/efficiency/complexity, gives a clear recommendation, and states when the "loser" is actually the right call.

What the interviewer is probing: Do you understand the difference between a client-pull request/response model and a server-push persistent-connection model, and can you justify a transport choice with concrete trade-offs (latency, wasted requests, scale, infra constraints) instead of just naming the trendy answer?

Clarifying assumptions: Real-time, bidirectional chat (send + instant receive), typing indicators and presence desired, moderate-to-large scale, standard browser + mobile clients. No exotic firewall constraints unless noted.

Plan: Define each option → evaluate against chat's needs → recommend WebSockets and explain precisely why → note the legitimate cases for polling/SSE → give the pragmatic hybrid.

The options on the table

Short polling — the client asks GET /messages?since=... on a fixed timer (say every 2s). Simple, works everywhere, but almost every request returns "nothing new," and any new message waits up to the interval before it's seen. Latency and wasted load are both bad.

Long polling — the client sends a request; the server holds it open until a message arrives (or a timeout), then responds; the client immediately re-requests. This gets near-real-time delivery over plain HTTP, but each message still costs a full request/response round trip plus reconnection, and it ties up server connections.

Server-Sent Events (SSE) — one long-lived HTTP connection over which the server streams events to the client. Efficient server→client push, but one-directional — the client still needs separate HTTP requests to send. Fine for feeds/notifications, awkward for two-way chat.

WebSocket — a single TCP connection, upgraded from HTTP once (Upgrade: websocket), that then stays open for full-duplex messaging: either side can send a frame at any time with almost no per-message overhead. Purpose-built for exactly this.

Why WebSockets win for chat

Chat has three properties that map directly onto WebSockets:

Bidirectional. Users both send and receive continuously. WebSocket is full-duplex over one connection; polling and SSE need separate mechanisms for one of the two directions.

Low latency is the product. A message should appear "instantly." With WebSockets the server pushes the moment a message arrives — no wait for a poll interval, no new-request setup. Short polling's latency is bounded by the interval; long polling adds a round trip per message.

Efficiency at scale. Short polling generates a storm of mostly-empty requests: 10k idle users polling every 2s = 5k req/s of pure waste, each paying TCP/TLS/HTTP header cost. A WebSocket is one idle open connection with a tiny periodic ping — dramatically less CPU, bandwidth, and header overhead. This is the decisive operational argument.

Concretely, a WebSocket frame carries ~2–14 bytes of framing vs the hundreds of bytes of HTTP headers on every polling request. For a high-traffic chat, that difference is the whole ball game.

When polling / SSE is actually the right call (shows judgment)
Very low interactivity / large idle populations where a bit of delay is fine — e.g. a dashboard that refreshes stats every 30s. Short polling is simpler and needs no special infra.
Restrictive environments — some corporate proxies, old load balancers, or serverless platforms don't handle long-lived upgraded connections well. Long polling degrades gracefully over plain HTTP and is the classic fallback (this is exactly why libraries like Socket.IO auto-fall-back from WebSocket to long polling).
One-way streams (live scores, notification feeds, log tailing) — SSE is simpler than WebSockets, auto-reconnects, and rides normal HTTP infra.
Operational simplicity / tiny scale — for a prototype with a handful of users, short polling every couple of seconds is perfectly acceptable and quicker to ship.
The pragmatic production answer

Use WebSockets as the primary transport, with long polling as an automatic fallback. That's what battle-tested stacks do: attempt the WebSocket upgrade; if the network/proxy blocks it, transparently drop to long polling so the user still gets messages. You pay the complexity (stateful connection tier, sticky routing or pub-sub for cross-server delivery, heartbeats/reconnection, backpressure) only because chat's bidirectional, low-latency nature genuinely demands it.

Cost of choosing WebSockets (be honest about it)

WebSockets aren't free: the connection tier is stateful, so you need sticky load balancing or a pub-sub layer to route a message to whichever server holds the recipient's socket; you must implement heartbeats/ping-pong to detect dead connections, reconnect with backoff on drop, and replay missed messages on reconnect (fetch since=lastSeq). Polling sidesteps all of this because every request is independent and stateless. This trade — stateful complexity for real-time efficiency — is the crux of the decision.

Summary / what a strong answer emphasizes

Lead with the model distinction: polling is client-pull request/response, WebSocket is a persistent full-duplex push channel. For chat, WebSockets are the right default because chat is bidirectional, latency-sensitive, and high-volume, and polling wastes requests and adds latency. Then show range: SSE for one-way pushes, long polling as the graceful fallback, short polling for low-interactivity or prototypes — and note the real cost of WebSockets (stateful tier, sticky/pub-sub routing, reconnection). Naming WebSockets with the fallback and the trade-offs is what makes the answer senior.

#### Common mistakes
saying "WebSockets, always" with no trade-offs; confusing SSE (one-way) with full-duplex; ignoring the wasted-request cost of short polling; forgetting reconnection/missed-message replay. Follow-up probes: "How do you keep a WebSocket alive through a proxy?" (ping/pong + timeouts), "How do you scale WebSockets across servers?" (pub-sub broker, sticky sessions), "What if the connection drops mid-conversation?" (reconnect + fetch since last seq).

**Comparison:**

```
Criterion	Short polling	Long polling	SSE	WebSocket
Direction	pull (req/res)	pull (held req)	server→client only	full-duplex
Latency	up to interval	~1 RTT/msg	low (server push)	lowest (push)
Wasted requests	many (mostly empty)	one req per msg	none	none
Connection	new each poll	held, then re-open	1 long-lived HTTP	1 long-lived TCP
Per-msg overhead	full HTTP headers	full HTTP headers	small event frame	tiny WS frame
Server state	stateless	ties up a conn	streaming conn	stateful socket
Send from client	yes	yes	needs separate HTTP	yes (same conn)
Best fit	prototypes, low-interactivity	HTTP-only fallback	notifications/feeds	chat
```

**Code:**

```
JS
Copy
// SHORT POLLING (client) — simple but wasteful + laggy
setInterval(async () => {
  const res = await fetch(`/api/messages?since=${lastSeq}`);
  const msgs = await res.json();          // usually [] -> wasted round trip
  for (const m of msgs) { render(m); lastSeq = m.seq; }
}, 2000);                                  // up to 2s latency per message

// WEBSOCKET (client) — push, full-duplex, reconnect + replay
function connect() {
  const ws = new WebSocket('wss://chat.example.com/ws?token=' + jwt);
  ws.onopen    = () => ws.send(JSON.stringify({ type: 'resume', since: lastSeq }));
  ws.onmessage = e => { const m = JSON.parse(e.data); render(m); lastSeq = m.seq; };
  ws.onclose   = () => setTimeout(connect, backoff());   // reconnect w/ backoff
  // send instantly, any time (full-duplex):
  sendBtn.onclick = () => ws.send(JSON.stringify({ type: 'send', body: input.value }));
  // heartbeat so dead sockets are detected
  setInterval(() => ws.readyState === 1 && ws.send('{"type":"ping"}'), 15000);
}
connect();

// WEBSOCKET (server, Node ws) — server pushes to whoever is connected
wss.on('connection', (ws, req) => {
  const userId = auth(req);
  sockets.set(userId, ws);                 // stateful: this server holds this socket
  ws.on('message', raw => chat.handle(userId, JSON.parse(raw)));
  ws.on('close', () => sockets.delete(userId));
});
```

**Example input / output:**

```text
SHORT POLLING (interval = 2s)
  t=0.0s  GET /messages?since=41 -> []          (wasted)
  t=2.0s  GET /messages?since=41 -> []          (wasted)
  t=2.3s  Bob's message arrives on server...
  t=4.0s  GET /messages?since=41 -> [seq 42]    -> delivered ~1.7s late

WEBSOCKET
  t=2.3s  server: ws.send({seq:42, body:"hi"}) -> client renders in <50ms
          (no polling loop, no empty requests, either side can send anytime)
```

**Flow:**

1. no, periodic refresh ok
2. yes
3. one-way push only
4. two-way, low latency, highvolume
5. yes
6. no
7. Need real-time chat?
8. Short polling — simplest
9. Bidirectional?
10. SSE
11. WebSocket — primary
12. Proxy/infra blocks upgrade?
13. Fall back to Long polling
14. Use WebSocket

