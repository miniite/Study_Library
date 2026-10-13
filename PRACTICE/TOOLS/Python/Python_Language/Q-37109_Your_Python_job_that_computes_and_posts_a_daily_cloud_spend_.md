---
type: "practice"
difficulty: "Intermediate"
---

# Your Python job that computes and posts a daily cloud spend report is slow because it re-fetches everything each run. How would you introduce incremental fetching and a local cache with correct invalidation?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Strategies](<../../../../TOOLS/Python/Python_Language/Cache_Invalidation_Strategies/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the performance issue of your Python job that computes and posts a daily cloud spend report, we need to implement an incremental approach combined with a local cache system that correctly invalidates stale data. This involves fetching only what has changed since the last run instead of re-fetching everything each time. Here’s how you can do it:

#### Triage
The core issue is that your current job is inefficient because it fetches all cloud spend data every day, leading to unnecessary processing and potential performance bottlenecks. We need to introduce a cache mechanism that stores the most recent data locally and only updates it when necessary.

#### First Principles Solution
Use a local database (like SQLite) or file-based storage to store the fetched data with timestamps.
Implement an incremental fetch logic that checks for new or updated spend entries since the last run using timestamps stored in the cache.
Periodically invalidate and refresh the cache when certain conditions are met, such as changes in billing periods.

Let's walk through this step-by-step:

Setup Local Cache: Use a simple SQLite database to store fetched cloud spend data along with a timestamp indicating when it was last updated.
Incremental Fetch Logic:
Query the cache for the latest update timestamp.
Fetch spend data from the cloud API and compare timestamps to determine which records are new or have been updated.
Only store these updates in the cache.
Cache Invalidation: Determine when to invalidate the cache, e.g., at the start of a new billing period.

Here's a basic implementation outline:

PYTHON
import sqlite3
from datetime import datetime

# Define database and table schema
DATABASE = 'cache.db'
TABLE_NAME = 'cloud_spend'

def init_db():
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    query = f'''
        CREATE TABLE IF NOT EXISTS {TABLE_NAME} (
            id INTEGER PRIMARY KEY,
            timestamp TEXT,
            data TEXT
        )
    '''
    cursor.execute(query)
    conn.commit()
    conn.close()

def fetch_and_update_spend(cache_timestamp):
    # Connect to the database
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()

    # Fetch new spend entries since the last cache update
    query = f'''
        SELECT * FROM {TABLE_NAME} WHERE timestamp > ?
    '''
    cursor.execute(query, (cache_timestamp,))
    new_entries = cursor.fetchall()

    for entry in new_entries:
        # Process each new or updated entry here
        print(f"New/Updated spend: {entry}")

    conn.close()

def main():
    init_db()

    # Simulate fetching the latest cache timestamp
    cache_timestamp = '2023-10-01 12:00:00'

    # Fetch and update spend data based on the cache timestamp
    fetch_and_update_spend(cache_timestamp)

if __name__ == "__main__":
    main()

#### Closing Summary
+ Common Mistakes/Follow-Ups:

This approach ensures that you only process new or updated cloud spend entries, significantly reducing the processing time. Some common mistakes to avoid include not properly handling edge cases like network failures during data fetches and ensuring correct timestamp comparisons in the cache.

For follow-ups, consider adding robust error handling, logging for debugging purposes, and possibly integrating a more sophisticated caching mechanism if you expect high volumes of data or frequent updates.

