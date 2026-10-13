---
type: "practice"
difficulty: "Deep"
---

# Design a file upload feature that supports resuming interrupted large uploads.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent Order APIs](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a file upload feature with support for resuming interrupted large uploads, we need to ensure both robustness and efficiency in handling interruptions without compromising performance or user experience. At a high level, our goal is to design a system that can detect where an interruption has occurred and resume the upload from that point. This involves several key components: storage management, metadata tracking, and efficient resumption logic.

#### Triage

The question asks us to design a feature for handling large file uploads with resumable support. The core requirements here are:

Resilience: Ability to handle interruptions.
Efficiency: Minimize overhead when resuming the upload.
Scalability: Support for large files and high volumes of concurrent uploads.

#### First-Principles Solution

To achieve this, we can break down the solution into several steps:

Metadata Storage:
Store metadata about each file, including its total size, chunk sizes, and a list of uploaded chunks.
Chunk-Based Uploads:
Divide files into fixed-size chunks for easier management and resumption.
Resumption Logic:
Implement logic to detect the last uploaded chunk and resume from there if an interruption is detected.
Detailed Design
1. Metadata Storage

We will store metadata about each file in a database, which includes the total file size, the chunk size, and a list of completed chunks with their respective offsets and timestamps. This information allows us to track the state of the upload and resume from the last successful chunk.

SQL
CREATE TABLE FileUploads (
    id INT PRIMARY KEY AUTO_INCREMENT,
    filename VARCHAR(255) NOT NULL,
    total_size BIGINT NOT NULL,
    chunk_size INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
SQL
CREATE TABLE Chunks (
    id INT PRIMARY KEY AUTO_INCREMENT,
    upload_id INT NOT NULL,
    offset BIGINT NOT NULL,  // Starting byte of the chunk
    data BLOB NOT NULL,      // The actual chunk data
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (upload_id) REFERENCES FileUploads(id)
);
2. Chunk-Based Uploads

When a user initiates an upload, we divide the file into chunks and send them one by one to our server. Each chunk is identified by its offset in the file.

PYTHON
def split_file_into_chunks(file_path, chunk_size):
    with open(file_path, 'rb') as file:
        while True:
            chunk = file.read(chunk_size)
            if not chunk:
                break
            yield (len(file.tell()), chunk)
3. Resumption Logic

Upon receiving a new upload, we first check the metadata to see which chunks have already been uploaded and only send or resume from the last completed chunk.

PYTHON
def handle_upload(upload_id):
    # Fetch metadata
    file_upload = get_file_upload_by_id(upload_id)

    # Get list of incomplete chunks
    incomplete_chunks = [chunk for chunk in get_chunks_for_file(upload_id) if not is_chunk_complete(chunk)]

    # Send/receive each chunk from the last complete one
    for offset, data in split_file_into_chunks(file_path, file_upload.chunk_size):
        if (offset - file_upload.total_size) < 0:
            send_chunk_to_server(chunk)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, our design ensures that large files can be uploaded with resumption support by dividing the file into chunks, storing metadata about the upload state, and efficiently resuming from where an interruption occurred. Key challenges include ensuring data integrity during interruptions and handling edge cases like partial uploads or network errors.

#### Common mistakes
to avoid include:

Not properly tracking chunk states in metadata.
Overhead in managing large numbers of small files/chunks.
Failing to handle partial uploads cleanly, potentially leading to corrupted files.

For follow-ups, we could discuss more advanced features such as parallel upload handling, multi-threaded downloads for faster resumption, or integrating with cloud storage solutions like S3 for better scalability and reliability.

**Comparison:**

```
Column Name	Data Type	Description
id	INT	Unique ID for each file upload.
filename	VARCHAR	Name of the uploaded file.
total_size	BIGINT	Total size of the file in bytes.
chunk_size	INT	Size of each chunk in bytes.
created_at	TIMESTAMP	Timestamp when the upload was initiated.
updated_at	TIMESTAMP	Timestamp for the last update.
Column Name	Data Type	Description
id	INT	Unique ID for each chunk.
upload_id	INT	Foreign key referencing FileUploads(id).
offset	BIGINT	Starting byte of the chunk within the file.
data	BLOB	Actual binary data of the chunk.
created_at	TIMESTAMP	Timestamp when the chunk was uploaded.
```

**Code:**

```python
def split_file_into_chunks(file_path, chunk_size):
    with open(file_path, 'rb') as file:
        while True:
            chunk = file.read(chunk_size)
            if not chunk:
                break
            yield (len(file.tell()), chunk)

def handle_upload(upload_id):
    # Fetch metadata
    file_upload = get_file_upload_by_id(upload_id)

    # Get list of incomplete chunks
    incomplete_chunks = [chunk for chunk in get_chunks_for_file(upload_id) if not is_chunk_complete(chunk)]

    # Send/receive each chunk from the last complete one
    for offset, data in split_file_into_chunks(file_path, file_upload.chunk_size):
        if (offset - file_upload.total_size) < 0:
            send_chunk_to_server(chunk)
```

**Example input / output:**

```text
Input:
- User initiates upload for a large file.
- Metadata is stored with the total size and initial chunks uploaded.

Output:
- Server stores metadata about the file upload, including completed chunk offsets.
- Upon resuming, server checks the last complete chunk and sends/receives subsequent chunks from that point onward.
```

**Data:**

```sql
CREATE TABLE FileUploads (
    id INT PRIMARY KEY AUTO_INCREMENT,
    filename VARCHAR(255) NOT NULL,
    total_size BIGINT NOT NULL,
    chunk_size INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE Chunks (
    id INT PRIMARY KEY AUTO_INCREMENT,
    upload_id INT NOT NULL,
    offset BIGINT NOT NULL,  // Starting byte of the chunk
    data BLOB NOT NULL,      // The actual chunk data
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (upload_id) REFERENCES FileUploads(id)
);
```

**Flow:**

1. Yes
2. No
3. Initiate Upload
4. Is File Already Uploaded?
5. Fetch Metadata
6. Resume Upload from LastCompleted Chunk
7. Divide File into Chunks
8. Upload Chunks Sequentially orParallelize
9. Store Chunk Data andMetadata

