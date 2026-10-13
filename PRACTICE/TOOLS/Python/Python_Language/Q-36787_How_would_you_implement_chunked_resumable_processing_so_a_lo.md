---
type: "practice"
difficulty: "Deep"
---

# How would you implement chunked, resumable processing so a long ETL job can restart from a checkpoint after a crash without reprocessing everything?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Data Migrations in Python](<../../../../TOOLS/Python/Python_Language/Large_Data_Migrations_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement chunked, resumable processing in Python, we need to ensure that each chunk of data is processed independently and can be restarted from where it left off in case of an interruption or crash. The key idea here is to maintain a state that allows us to track the progress of each chunk and resume from this point if necessary.

The primary steps involve:

Dividing the Data: Splitting the input data into manageable chunks.
Tracking Progress: Maintaining a record of completed chunks.
Resuming Processing: Restarting processing from the last known checkpoint.
Handling Failures Gracefully: Ensuring that failures are handled and recovery is straightforward.

Let's break down these steps in detail:

#### Triaging the Question

The interviewer is likely probing into your understanding of:

How to handle stateful operations in Python, especially when dealing with large datasets or long-running processes.
The trade-offs between performance and fault tolerance.
Best practices for maintaining consistency and reliability.

#### Assumptions
The data can be processed in chunks.
Each chunk is independent and can be processed without dependencies on previous chunks (except for initial setup).
There's a mechanism to save the progress, such as writing checkpoints to disk or a database.
The processing function is idempotent.

#### First-Principles Solution
Chunking Data: We start by dividing the input data into fixed-size chunks using a generator function.
Progress Tracking: Each chunk has an identifier (e.g., file name, sequence number) and a progress status (processed or not).
Processing Function: A stateful processing function that reads each chunk, processes it, and updates its state.
Checkpointing: After processing a chunk, we write the updated state to a checkpoint file.

Here's an example implementation:

PYTHON
import os

def process_chunk(chunk_data, chunk_id):
    # Simulate processing logic
    print(f"Processing chunk {chunk_id}...")

def save_checkpoint(chunk_id, processed=False):
    with open("checkpoint.txt", "a") as f:
        f.write(f"{chunk_id},{processed}\n")

def load_checkpoints():
    checkpoints = {}
    if os.path.exists("checkpoint.txt"):
        with open("checkpoint.txt", "r") as f:
            for line in f:
                chunk_id, processed_str = line.strip().split(",")
                processed = bool(int(processed_str))
                checkpoints[int(chunk_id)] = (int(chunk_id), processed)
    return checkpoints

def process_data(data):
    # Split data into chunks
    chunk_size = 1024  # Adjust based on your needs
    for i in range(0, len(data), chunk_size):
        chunk = data[i:i + chunk_size]
        yield (i // chunk_size, chunk)

# Example usage
if __name__ == "__main__":
    # Simulated large dataset
    data = [f"Data {i}" for i in range(100)]

    # Load existing checkpoints
    checkpoints = load_checkpoints()

    # Process each chunk
    for chunk_id, chunk in process_data(data):
        if chunk_id not in checkpoints or not checkpoints[chunk_id][1]:
            process_chunk(chunk, chunk_id)
            save_checkpoint(chunk_id, processed=True)

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that the processing can be resumed from any point of failure. However, there are a few common pitfalls:

Chunk Size: Choosing an optimal chunk size is crucial; too large might overwhelm memory, while too small might be inefficient.
Synchronization Issues: Ensure that state updates (like checkpoints) are written atomically to avoid inconsistencies.
Error Handling: Implement robust error handling to ensure that the processing can recover gracefully from transient errors.

#### Follow-Ups
How would you optimize the chunk size for large datasets?
Can you discuss how this approach would scale in a distributed environment?
What measures would you take to make sure checkpointing is efficient?

**Code:**

```python
import os

def process_chunk(chunk_data, chunk_id):
    # Simulate processing logic
    print(f"Processing chunk {chunk_id}...")

def save_checkpoint(chunk_id, processed=False):
    with open("checkpoint.txt", "a") as f:
        f.write(f"{chunk_id},{processed}\n")

def load_checkpoints():
    checkpoints = {}
    if os.path.exists("checkpoint.txt"):
        with open("checkpoint.txt", "r") as f:
            for line in f:
                chunk_id, processed_str = line.strip().split(",")
                processed = bool(int(processed_str))
                checkpoints[int(chunk_id)] = (int(chunk_id), processed)
    return checkpoints

def process_data(data):
    # Split data into chunks
    chunk_size = 1024  # Adjust based on your needs
    for i in range(0, len(data), chunk_size):
        chunk = data[i:i + chunk_size]
        yield (i // chunk_size, chunk)

# Example usage
if __name__ == "__main__":
    # Simulated large dataset
    data = [f"Data {i}" for i in range(100)]

    # Load existing checkpoints
    checkpoints = load_checkpoints()

    # Process each chunk
    for chunk_id, chunk in process_data(data):
        if chunk_id not in checkpoints or not checkpoints[chunk_id][1]:
            process_chunk(chunk, chunk_id)
            save_checkpoint(chunk_id, processed=True)
```

