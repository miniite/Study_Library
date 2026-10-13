---
type: "practice"
difficulty: "Deep"
---

# How would you implement cursor/keyset pagination for an infinite-scroll frontend in Python, and why does offset pagination degrade and produce duplicates under writes?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming Query Results](<../../../../TOOLS/Python/Python_Language/Streaming_Query_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement cursor/keyset pagination for an infinite-scroll frontend using Python, we focus on leveraging a unique identifier (cursor) to fetch subsequent pages of data without ambiguity or duplication issues that can arise from offset-based pagination. In this approach, the server uses the provided cursor to determine where to start fetching new records, ensuring a more reliable and performant experience for the user.

#### Triaging the Question

The question primarily asks us to explain how to implement cursor/keyset pagination using Python in an infinite-scroll frontend scenario. This involves understanding the mechanism of cursor-based pagination and its advantages over offset pagination, particularly focusing on why offset pagination can degrade and produce duplicates under write operations.

Key Assumptions
The backend is written in Python.
We are dealing with a large dataset that needs to be paginated for display in an infinite-scroll frontend.
The application handles multiple concurrent writes without conflicts.
Step-by-Step Solution From First Principles

Understand the Problem: Offset-based pagination works by fetching records from a specific starting point (offset) and a fixed number of subsequent records. This approach can become inefficient when dealing with large datasets or frequent updates, as it requires scanning through all preceding records to find the desired offset.

Cursor/Keyset Pagination Mechanics:

When an infinite-scroll request is made by the frontend, the client sends a cursor value (often a primary key or unique identifier of the last item in the previous page).
The backend uses this cursor to fetch the next set of records starting from that specific point.

Implementing Cursor-based Pagination:

Store the data in a database with an auto-incrementing primary key.
On receiving a request, extract the cursor value.
Use a query that starts fetching records immediately after the last item indicated by the cursor to ensure no duplicates are returned.
Return a new cursor for the next page.

Here’s how you might implement this in Python using Flask and SQLAlchemy:

PYTHON
from flask import Flask, request
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base

app = Flask(__name__)
Base = declarative_base()

class Item(Base):
    __tablename__ = 'items'
    id = Column(Integer, primary_key=True)
    data = Column(String)

Session = sessionmaker(bind=engine)  # Assuming engine is already set up

@app.route('/items/<cursor>')
def get_items(cursor=None):
    if cursor:
        last_item_id = int(cursor)
        next_items = session.query(Item).filter(Item.id > last_item_id).limit(10).all()
        new_cursor = str(next_items[-1].id) if next_items else None
        return {'items': [item.data for item in next_items], 'cursor': new_cursor}
    else:
        initial_items = session.query(Item).limit(10).all()
        first_item_id = str(initial_items[-1].id)
        return {'items': [item.data for item in initial_items], 'cursor': first_item_id}

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes/Follow-Ups

Cursor-based pagination ensures that each page is fetched accurately without overlap or redundancy, making it ideal for infinite-scroll applications. The main challenge lies in the implementation细节，如确保cursor的唯一性和正确处理边界情况。常见的错误包括：

忽略了事务的一致性问题，导致在并发写入时数据不一致。
没有正确地管理 cursor 的范围，可能导致查询范围过大或过小。

在实际应用中，还需要考虑性能优化、缓存机制以及高可用性的实现策略。同时，对于更深层次的讨论可以涉及数据库设计和事务处理等高级话题。

