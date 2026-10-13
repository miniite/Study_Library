---
type: "practice"
difficulty: "Expert"
---

# Design the architecture for a design-collaboration canvas (Figma-scale) that must render a 100K-object document at 60fps in the browser and sync multiplayer edits — how do you split the rendering engine (WebGL/WASM) from the sync layer and keep memory bounded?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-region Write Paths](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Cross-region_Write_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an architecture that can handle a 100K-object document at 60fps in the browser while synchronizing multiplayer edits, we need to split the rendering engine from the sync layer and ensure memory usage is bounded. This requires a strategic approach considering both the performance and synchronization needs of the application.

#### Triaging the Question

The question asks for an architecture design that can handle:

Rendering 100K objects at 60fps in the browser.
Synchronizing multiplayer edits efficiently.
Ensuring memory usage is managed to avoid excessive consumption.
Key Assumptions and Probing Areas
WebGL/WASM: WebGL for rendering, WASM (WebAssembly) for performance-critical logic.
Memory Management: Bounded memory usage to prevent crashes or excessive resource consumption.
Synchronization: Real-time updates across multiple clients without overwhelming the server.
Step-by-Step Solution From First Principles

Rendering Engine (WebGL/WASM)

WebGL for Rendering: Use WebGL to handle the heavy lifting of rendering 3D objects, which is well-suited for complex graphics.
WASM for Logic: Utilize WebAssembly for performance-critical logic like physics or calculations, as it can offer significant performance improvements compared to JavaScript.

Multiplayer Synchronization Layer

Server-Side Logic: Implement server-side logic using a reliable protocol (e.g., gRPC) and a state synchronization model.
Client-Side Updates: Clients send updates to the server in real-time, and the server maintains a consistent state that is then sent back to all clients.

Memory Management

Object Pooling: Implement object pooling to reuse objects instead of creating new ones frequently.
Garbage Collection Optimization: Optimize garbage collection by ensuring objects are marked for removal correctly and periodically cleaning up unused objects.

#### Performance Considerations

Batch Rendering: Batch rendering operations to minimize the number of draw calls in WebGL, which is critical for performance with large numbers of objects.
Async Loading: Use asynchronous loading techniques to load objects only as they come into view or are needed, reducing initial memory footprint.
Detailed Architecture

Client-Side

WebGL Canvas
Render Loop
Object Pooling
Batch Rendering
WebAssembly Logic
DOM Updates

Server-Side

Server
State Synchronization Protocol
Client Updates

#### Cons
istent State
Client Updates

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By splitting the rendering engine into WebGL and WebAssembly, and implementing a robust synchronization layer with server-side state management, we can achieve real-time collaboration while maintaining performance and bounded memory usage.

#### Common Mistakes
Failing to implement object pooling or garbage collection optimization, leading to excessive memory consumption.
Not using batch rendering in WebGL, resulting in frequent draw calls that degrade performance.
Overloading the server with too many updates, causing latency issues.
Further Discussion
How do you handle edge cases like network latency and disconnections?
What strategies can be used for optimizing rendering further?
How would you implement security measures to prevent malicious edits?

**Comparison:**

```
Strategy	Description
Object Pooling	Reuse objects instead of creating new ones.
Batch Rendering	Minimize draw calls to improve performance.
```

**Code:**

```javascript
// Example of WebAssembly initialization
const module = new WebAssembly.Module(fetch('path/to/module.wasm').then(r => r.arrayBuffer()));
const instance = new WebAssembly.Instance(module);
instance.exports.init();
```

**Example input / output:**

```text
Input:
- Object data (up to 100K objects)
- Multiplayer updates

Output:
- Rendered canvas with 100K objects at 60fps
- Synchronized edits across all clients
```

**Data:**

```json
{
  "objects": [
    {"id": 1, "type": "rectangle", "position": [0, 0]},
    // ... up to 100K objects
  ]
}
```

**Flow:**

1. Client
2. WebGL Canvas
3. Batch Rendering
4. Update Render List
5. WebAssembly Logic
6. DOM Updates
7. Server
8. State Synchronization Protocol
9. Client Updates
10. Consistent State
11. Client Updates

