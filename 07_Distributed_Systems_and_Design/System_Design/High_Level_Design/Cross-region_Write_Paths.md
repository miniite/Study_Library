---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Cross-region_Write_Paths"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["AWS"]
---

# Cross-region Write Paths

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [For a collaborative editor's frontend, how do you render remote cursors, apply incoming ops smoothly, and keep the local doc responsive while syncing over WebSocket?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29630_For_a_collaborative_editor_s_frontend_how_do_you_render_remo.md>)
- [Deep] [Design a read-heavy service with the CQRS pattern where writes go to a normalized store and reads to a denormalized projection — how do you keep the projection eventually consistent and handle projection rebuild?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29743_Design_a_read-heavy_service_with_the_CQRS_pattern_where_writ.md>)
- [Deep] [You're deciding whether to run a strong-consistency database everywhere for simplicity or mix in eventually-consistent stores for scale — a bar-raiser asks you to defend the operational complexity of a mixed model; do it.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29874_You_re_deciding_whether_to_run_a_strong-consistency_database.md>)
- [Deep] [You must architect the read-replica-and-consistency strategy for a product where users must see their own edits instantly but stale reads are fine for others — how do you route sticky reads use write-tokens and keep the fast path off the primary?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29935_You_must_architect_the_read-replica-and-consistency_strategy.md>)
- [Expert] [Design a collaborative document editor's backend (Google-Docs-style) — how do you handle concurrent edits, conflict resolution (OT vs CRDT), and offline reconciliation?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29629_Design_a_collaborative_document_editor_s_backend_Google-Docs.md>)
- [Expert] [Design the multi-region write-conflict-resolution strategy for user-editable data where LWW would silently lose edits — evaluate CRDTs, app-level merge, and single-writer-per-key at organizational scale.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29836_Design_the_multi-region_write-conflict-resolution_strategy_f.md>)
- [Expert] [Architect the write-path for a system where writes must be durable across regions before acknowledgment but latency budgets forbid a synchronous cross-region quorum — how do you resolve the tension?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29873_Architect_the_write-path_for_a_system_where_writes_must_be_d.md>)
- [Expert] [Design the architecture for a design-collaboration canvas (Figma-scale) that must render a 100K-object document at 60fps in the browser and sync multiplayer edits — how do you split the rendering engine (WebGL/WASM) from the sync layer and keep memory bounded?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29908_Design_the_architecture_for_a_design-collaboration_canvas_Fi.md>)

## Related concepts

- [Multi-tenant Migrations](<../System_Design_Core/Read-through_and_Write-behind_Caching/Multi-tenant_Migrations.md>) *(System Design Core)*
- [Distributed Coordination Choices](<../../Distributed_Systems/Concurrency_Patterns/Distributed_Coordination_Choices.md>) *(Concurrency Patterns)*
- [Cross-service Invalidation Races](<../../Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>) *(Concurrency Patterns)*

## References

- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — A (Book) Case for Eventual Consistency A Short Story of Keeping Inventory at a Bookstore to Explain
- *Ultimate AWS Data Engineering* — Rathish Mohan — Architecting for Scalability Data Replication Techniques
- *Unifying Business, Data, and Code* — Ron Itelman, Juan Cruz Viotti — The Science of Synchronization

