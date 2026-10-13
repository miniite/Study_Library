---
type: "practice"
difficulty: "Expert"
---

# Design a collaborative document editor's backend (Google-Docs-style) — how do you handle concurrent edits, conflict resolution (OT vs CRDT), and offline reconciliation?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-region Write Paths](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Cross-region_Write_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design the backend of a Google Docs-style collaborative document editor, we need to focus on handling concurrent edits efficiently, ensuring data consistency through conflict resolution mechanisms, and managing user sessions in an offline environment. At a high level, our goal is to create a robust system that can handle multiple users editing the same document simultaneously without losing data integrity or causing data loss.

We'll start by defining the core requirements and constraints of our design:

Concurrency Control: Handle simultaneous edits from different users.
Conflict Resolution (CR): Resolve conflicts when edits overlap.
Offline Reconciliation: Ensure consistency when users come back online after being disconnected.

Let's break down each component in detail:

1. Data Model and Storage

We need a robust data model to store the document content efficiently. A Content Addressable Storage (CAS) approach can be used, where each piece of text is stored by its unique hash. This ensures that identical texts are not duplicated.

PLAINTEXT
Document {
    id: string,
    version: number,
    chunks: [Chunk],
}

Chunk {
    start_offset: number,
    end_offset: number,
    content_hash: string,
}
2. Concurrency Control with Optimistic Locking (OT)

For concurrent edits, we can use Optimistic Locking (OT) to minimize conflicts:

Each user's editor maintains a local copy of the document.
When a user wants to save changes, they send their local version back to the server along with a version number.
The server checks if the received version matches the current version in storage. If not, it indicates a conflict.
PLAINTEXT
Request: SaveDocumentRequest {
    doc_id: string,
    version: number,
    chunks: [Chunk],
}

Response: SaveDocumentResponse {
    success: bool,
    conflicts: [Chunk],
}
3. Conflict Resolution (CRDTs)

To handle conflicts, we use Conflict-Free Replicated Data Types (CRDTs) like the Vector Clock or Lamport Timestamps:

Each edit operation is tagged with a vector clock representing the history of all operations.
When conflicts arise, the server resolves them based on the timestamps. For example, if two edits overlap at the same position, the most recent edit wins.
4. Offline Reconciliation

For users who go offline, we need to ensure their local changes are merged back into the main document when they come online:

Delta Encoding: When a user comes online, send only the differences (deltas) between the current version and the last known version.
Rebase Strategy: The server re-applies these deltas on top of the latest version stored in its database.
5. High-Level Design Summary

Here’s a high-level flow for handling concurrent edits:

Users edit locally and send periodic updates or final saves to the server.
Server applies OT rules to check for conflicts.
If conflicts are detected, CRDT logic resolves them.
Offline users’ changes are reconciled with the latest version when they come online.

#### Common Mistakes
Ignoring Conflict Handling: Failing to implement proper conflict resolution can lead to data loss or inconsistencies.
Overcomplicating OT Logic: Overly complex OT rules can introduce bugs and performance issues.
Poor Offline Reconciliation Strategy: Inadequate delta encoding or rebase strategies can result in inefficient use of bandwidth and server resources.

