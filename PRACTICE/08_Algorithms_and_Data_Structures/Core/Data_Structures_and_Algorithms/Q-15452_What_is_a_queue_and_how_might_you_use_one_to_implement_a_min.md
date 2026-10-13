---
type: "practice"
difficulty: "Intermediate"
---

# What is a queue and how might you use one to implement a mini-batch data loader that prefetches batches?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Queues and Stacks](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Queues_and_Stacks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A queue is a linear data structure that follows the First-In-First-Out (FIFO) principle, where elements are added at the rear and removed from the front. In this context, we can use a queue to implement a mini-batch data loader that prefetches batches of data for training machine learning models. The core idea is to efficiently manage multiple data loading tasks so that as soon as one batch finishes processing, another is immediately available.

The question here involves understanding how to set up a data pipeline where each mini-batch can be processed concurrently while ensuring that the model always has fresh input batches. This requires managing multiple threads or processes and efficient memory management. Let's break it down step-by-step from first principles:

Understanding the Requirement: We need a mechanism to handle concurrent data loading, where each mini-batch is loaded as soon as one finishes being processed.
Choosing Data Structures and Algorithms: A queue is ideal for managing batches because it ensures that once a batch is ready (loaded), it can be immediately processed without waiting.

#### Implementation
Plan:
Create multiple threads or processes to load data into mini-batches.
Use the queue module in Python to manage these batches efficiently.
Step-by-Step Solution:
Initialize a queue with sufficient capacity for holding multiple mini-batches.
Spawn worker threads/processes that continuously add new batches to the queue.
In the main thread, dequeue and process each batch as soon as it is available.

Here’s an example implementation in Python:

PYTHON
import queue
import threading

class DataLoader:
    def __init__(self, num_workers=4, batch_size=32):
        self.queue = queue.Queue(maxsize=num_workers)
        self.num_workers = num_workers
        self.batch_size = batch_size
        self.workers = []

    def start(self):
        # Start worker threads/processes
        for _ in range(self.num_workers):
            t = threading.Thread(target=self.worker, daemon=True)
            t.start()
            self.workers.append(t)

    def worker(self):
        while True:
            batch = self.load_batch()  # Simulate data loading
            if batch is None:
                break
            self.queue.put(batch)  # Enqueue the loaded batch

    def load_batch(self):
        # Simulated function to fetch a new batch of data
        pass

    def get_next_batch(self):
        return self.queue.get()  # Dequeue and process the next batch

# Example usage:
data_loader = DataLoader(num_workers=4, batch_size=32)
data_loader.start()

for _ in range(10):  # Process first 10 batches
    print(data_loader.get_next_batch())

This code demonstrates a simple setup using threads to load data into a queue. The key points are:

Efficient use of threading for concurrent loading.
Queue management to ensure FIFO processing.

**Comparison:**

```
Batch ID	Worker ID
1	2
2	4
3	3
...	...
Worker Thread	Status
1	Running
2	Running
3	Running
4	Running
```

**Code:**

```python
import queue
import threading

class DataLoader:
    def __init__(self, num_workers=4, batch_size=32):
        self.queue = queue.Queue(maxsize=num_workers)
        self.num_workers = num_workers
        self.batch_size = batch_size
        self.workers = []

    def start(self):
        for _ in range(self.num_workers):
            t = threading.Thread(target=self.worker, daemon=True)
            t.start()
            self.workers.append(t)

    def worker(self):
        while True:
            batch = self.load_batch()  # Simulate data loading
            if batch is None:
                break
            self.queue.put(batch)  # Enqueue the loaded batch

    def load_batch(self):
        pass

    def get_next_batch(self):
        return self.queue.get()
```

**Example input / output:**

```

Example input:

PYTHON
Copy
data_loader = DataLoader(num_workers=4, batch_size=32)
data_loader.start()

for _ in range(10):
    print(data_loader.get_next_batch())

Output (mocked for demonstration):

CODE
Copy
Batch 1 loaded
Batch 2 loaded
Batch 3 loaded
...
Batch 10 loaded
```

**Data:**

```json
{
    "num_workers": 4,
    "batch_size": 32
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is Queue Full?
7. Worker Thread Loads Batch
8. Enqueue Batch to Queue
9. Worker Thread Waits for NewBatch
10. Get Next Batch from Queue
11. Is Queue Empty?
12. End
13. Dequeue and Process Batch

