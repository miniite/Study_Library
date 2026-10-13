---
type: "practice"
difficulty: "Basic"
---

# How would you connect to a relational database from Python and run a SELECT query, and what does the DB-API 2.0 standard define for cursors and connections?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming Query Results](<../../../../TOOLS/Python/Python_Language/Streaming_Query_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To connect to a relational database from Python and run a SELECT query, you would typically use a Python library that adheres to the Database API Specification v2.0 (DB-API 2.0). This standard defines how Python programs interact with databases through connections and cursors.

The DB-API 2.0 standard is designed to provide a uniform interface for various database drivers, making it easier to switch between different database systems without changing your code significantly. The core components defined by this standard are Connection objects and Cursor objects.

