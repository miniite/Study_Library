---
type: "practice"
difficulty: "Deep"
---

# How would you handle partial failures when embedding a large batch of documents in a Flask ingestion job?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Error Handling](<../../../../TOOLS/Python/Flask/Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle partial failures in an ingestion job using Flask, we need to ensure that our system can process batches of documents robustly and recover gracefully from any issues that arise during processing. This involves designing a robust error handling mechanism, implementing retries, and ensuring that the application logs details effectively for debugging purposes.

#### Triage

The question is asking about how to manage partial failures in an ingestion job where large batches of documents are processed using Flask. The key aspects being probed include understanding error handling, dealing with concurrency, logging, and graceful recovery mechanisms.

#### Assumptions
We use a database to store the state of document processing.
Each document has a unique identifier.
We can retry failed operations up to a certain limit.
We log relevant information for debugging.

#### First Principles Solution
Database State Management: Use a database table to keep track of the documents that have been processed and their status (e.g., success, failure, pending).
Error Handling in Flask: Implement error handling at multiple levels: within each document processing function, during batch ingestion, and for external dependencies.
Concurrency Control: Handle concurrent access to the database and shared resources using locks or transactions.
Retry Mechanism: Implement a retry strategy with exponential backoff for failed operations.
Logging: Log detailed information about each document and its processing status.
Step-by-Step Solution

Define Document Processing Function

PYTHON
from flask import Flask, request
from sqlalchemy import create_engine, Column, Integer, String, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import time
import random

app = Flask(__name__)

Base = declarative_base()
engine = create_engine('sqlite:///documents.db')
Session = sessionmaker(bind=engine)

class Document(Base):
    __tablename__ = 'documents'

    id = Column(Integer, primary_key=True)
    content = Column(String)
    processed = Column(Boolean, default=False)

Base.metadata.create_all(engine)

def process_document(document_id):
    # Simulate document processing
    time.sleep(random.random() * 2)  # Simulate processing time

    if random.randint(0, 1) == 1:  # Random failure
        return False

    session = Session()
    doc = session.query(Document).filter_by(id=document_id).first()
    doc.processed = True
    session.commit()
    session.close()
    return True

Batch Ingestion Function

PYTHON
def batch_ingest(documents):
    session = Session()

    for document in documents:
        if not process_document(document['id']):
            # Mark the document as failed and log the error
            doc = session.query(Document).filter_by(id=document['id']).first()
            doc.processed = False
            doc.error_message = f"Failed to process {document['id']}"
            session.commit()

    session.close()

Error Handling in Flask Route

PYTHON
@app.route('/ingest', methods=['POST'])
def ingest():
    try:
        documents = request.get_json()
        batch_ingest(documents)
        return "Ingestion successful", 200
    except Exception as e:
        # Log the error and return a failure response
        app.logger.error(f"Error during ingestion: {str(e)}")
        return f"Failed to ingest documents due to an error: {str(e)}", 500

Retry Mechanism

PYTHON
from functools import wraps

def retry_with_backoff(attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt < attempts - 1:
                        time.sleep(delay * (2 ** attempt))
                    else:
                        raise
        return wrapper
    return decorator

@retry_with_backoff()
def process_document(document_id):
    # Simulate document processing with retries
    if random.randint(0, 1) == 1:  # Random failure
        raise Exception("Failed to process document")

    session = Session()
    doc = session.query(Document).filter_by(id=document_id).first()
    doc.processed = True
    session.commit()
    session.close()

Logging

PYTHON
import logging

app.logger.setLevel(logging.DEBUG)

def setup_logging():
    handler = logging.StreamHandler()
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    handler.setFormatter(formatter)
    app.logger.addHandler(handler)

setup_logging()

#### Closing Summary

By implementing a robust error handling mechanism, using retries with exponential backoff, and maintaining detailed logs, we can ensure that our Flask ingestion job handles partial failures gracefully. This solution involves tracking the state of documents in a database, processing them within a function, marking successes and failures, and retrying failed operations.

#### Common Mistakes/Follow-Ups
Not Handling Concurrency: Not managing concurrent access to shared resources like the database can lead to race conditions.
Insufficient Retries: Too few retries or no retries at all can result in loss of data.
Poor Logging: Lack of detailed logging makes it hard to diagnose and fix issues.

**Comparison:**

```
Document ID	Content	Processed
1	Document one	True
2	Document two	False
3	Document three	False
```

**Code:**

```python
from flask import Flask, request
from sqlalchemy import create_engine, Column, Integer, String, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import time
import random

app = Flask(__name__)

Base = declarative_base()
engine = create_engine('sqlite:///documents.db')
Session = sessionmaker(bind=engine)

class Document(Base):
    __tablename__ = 'documents'

    id = Column(Integer, primary_key=True)
    content = Column(String)
    processed = Column(Boolean, default=False)

Base.metadata.create_all(engine)

def process_document(document_id):
    time.sleep(random.random() * 2)  # Simulate processing time

    if random.randint(0, 1) == 1:  # Random failure
        return False

    session = Session()
    doc = session.query(Document).filter_by(id=document_id).first()
    doc.processed = True
    session.commit()
    session.close()
    return True

def batch_ingest(documents):
    session = Session()

    for document in documents:
        if not process_document(document['id']):
            # Mark the document as failed and log the error
            doc = session.query(Document).filter_by(id=document['id']).first()
            doc.processed = False
            doc.error_message = f"Failed to process {document['id']}"
            session.commit()

    session.close()

@app.route('/ingest', methods=['POST'])
def ingest():
    try:
        documents = request.get_json()
        batch_ingest(documents)
        return "Ingestion successful", 200
    except Exception as e:
        # Log the error and return a failure response
        app.logger.error(f"Error during ingestion: {str(e)}")
        return f"Failed to ingest documents due to an error: {str(e)}", 500

from functools import wraps

def retry_with_backoff(attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt < attempts - 1:
                        time.sleep(delay * (2 ** attempt))
                    else:
                        raise
        return wrapper
    return decorator

@retry_with_backoff()
def process_document(document_id):
    # Simulate document processing with retries
    if random.randint(0, 1) == 1:  # Random failure
        raise Exception("Failed to process document")

    session = Session()
    doc = session.query(Document).filter_by(id=document_id).first()
    doc.processed = True
    session.commit()
    session.close()

import logging

app.logger.setLevel(logging.DEBUG)

def setup_logging():
    handler = logging.StreamHandler()
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    handler.setFormatter(formatter)
    app.logger.addHandler(handler)

setup_logging()
```

**Example input / output:**

```text
Input:
{
  "documents": [
    {"id": 1, "content": "Document one"},
    {"id": 2, "content": "Document two"},
    {"id": 3, "content": "Document three"}
  ]
}

Output:
{
  "message": "Ingestion successful"
}
```

**Data:**

```sql
CREATE TABLE documents (
    id INTEGER PRIMARY KEY,
    content TEXT,
    processed BOOLEAN DEFAULT FALSE,
    error_message TEXT
);
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Start
6. Is document processed?
7. Mark as processed
8. Retry with backoff?
9. Log error and mark as failed
10. Retrying...
11. End

