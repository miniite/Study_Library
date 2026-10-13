---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between snapshot-based extraction and incremental (watermark) extraction from an OLTP source, and the failure modes of each on a table with soft deletes.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Feeding OLAP from OLTP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Feeding_OLAP_from_OLTP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between snapshot-based and incremental (watermark) extraction methods from an OLTP (Online Transaction Processing) system, we need to first grasp how these approaches handle data changes, particularly in tables with soft deletes. Snapshot-based extraction involves taking a complete copy of the relevant data at a given point in time, while incremental (watermark) extraction focuses on capturing only the changes that occur after the last known state.

Snapshot-based extraction is straightforward and provides a full snapshot of the table as it was at a specific timestamp. However, this approach can lead to issues when dealing with soft deletes, where records are marked for deletion rather than being physically removed from the table. On the other hand, incremental (watermark) extraction tracks changes more granularly, which is beneficial but introduces complexities in managing soft deletions.

Let's delve into the mechanics of both methods and their failure modes.

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Snapshot-based Extraction
6. Is timestamp recent?
7. Extract full snapshot
8. Re-extract with newertimestamp
9. Incremental (Watermark)Extraction
10. Is watermark current?
11. Extract changes since lastwatermark
12. Update watermark andre-extract

