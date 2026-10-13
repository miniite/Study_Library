---
type: "practice"
difficulty: "Deep"
---

# Model a class design for graceful shutdown of a service: stop accepting new work, drain in-flight requests, flush buffers, and honor a hard deadline; where do you hook the signal and how do you avoid dropping requests?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lock Granularity Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

In designing a graceful shutdown mechanism for a service, we aim to handle the process of stopping new work, draining in-flight requests, flushing buffers, and honoring a hard deadline. The key challenge is ensuring that no ongoing or pending requests are dropped during this process. We will design our solution by hooking into system signals to initiate the shutdown, and then implementing specific hooks within the service logic to manage these tasks.

#### Triaging the Question

The question is asking us to model a class design for graceful shutdown of a service. The interviewer is probing our understanding of low-level design principles, signal handling, and robust error management in critical system processes. We need to consider how signals are used to initiate the shutdown process and ensure that all ongoing operations are properly managed.

#### Assumptions
System Signals: We assume that the operating system provides a mechanism for sending signals (e.g., SIGINT or SIGTERM) to gracefully request the termination of processes.
Service State Management: The service maintains state information about current and pending tasks, buffers, etc.
Hard Deadline: There is a specific time by which all in-flight requests must be completed.

#### First-Principles Solution

Signal Handling:

We will define a signal handler that listens for shutdown signals from the operating system.
Upon receiving the signal, we initiate the graceful shutdown process.

Graceful Shutdown Process:

Stop accepting new work: This involves blocking new incoming requests.
Drain in-flight requests: Ensure all currently running tasks complete before proceeding further.
Flush buffers: Clear any pending data that needs to be sent out before shutting down.

Class Design:

PYTHON
class GracefulShutdownService:
    def __init__(self):
        self.shutdown_requested = False
        self.in_flight_requests = 0
        self.buffered_data_count = 0

    def handle_signal(self, signal_number, frame):
        print(f"Received signal {signal_number}. Initiating graceful shutdown.")
        self.shutdown_requested = True

    def accept_new_work(self, task):
        if not self.shutdown_requested:
            # Process and enqueue the new task
            pass
        else:
            raise ShutdownException("Service is shutting down. Please retry later.")

    def process_request(self, request):
        if not self.shutdown_requested:
            # Process the request
            self.in_flight_requests += 1
        else:
            print(f"Skipping request: Service is shutting down.")
            return False

    def send_buffered_data(self):
        while self.buffered_data_count > 0:
            data = get_next_buffered_data()
            if not self.shutdown_requested:
                # Send the buffered data
                pass
            else:
                break

#### Implementation

Hook into the signal handling mechanism using a library like signal in Python.
Ensure that critical sections of code are properly synchronized to avoid race conditions.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Avoiding Dropped Requests: By ensuring proper synchronization and blocking new requests, we can prevent dropping any ongoing or pending tasks. Properly managing the state during shutdown is crucial.
Handling Hard Deadline: Implement a timer that checks if the hard deadline has been reached. If so, force a shutdown even if in-flight requests are still pending.

#### Common Mistakes
Improper Signal Handling: Not correctly setting up signal handlers can lead to missed shutdown signals or improper handling of termination.
Ignoring Buffer State: Failing to flush buffers properly can result in data loss.
Race Conditions: Synchronization issues between threads handling requests and the main thread managing the shutdown process.

#### Follow-Ups
How would you ensure that the service remains responsive during the shutdown process?
What additional logging mechanisms would you implement to trace the state of the service during a graceful shutdown?

**Comparison:**

```
Signal	Action
SIGTERM	Graceful shutdown initiated
SIGKILL	Immediate termination without cleanup
```

**Code:**

```python
import signal

class ShutdownException(Exception):
    pass

class GracefulShutdownService:
    def __init__(self):
        self.shutdown_requested = False
        self.in_flight_requests = 0
        self.buffered_data_count = 0
        self.signal_handler_installed = False

    def handle_signal(self, signal_number, frame):
        print(f"Received signal {signal_number}. Initiating graceful shutdown.")
        self.shutdown_requested = True

    def install_signal_handler(self):
        if not self.signal_handler_installed:
            signal.signal(signal.SIGTERM, self.handle_signal)
            self.signal_handler_installed = True

    def accept_new_work(self, task):
        if not self.shutdown_requested:
            # Process and enqueue the new task
            pass
        else:
            raise ShutdownException("Service is shutting down. Please retry later.")

    def process_request(self, request):
        if not self.shutdown_requested:
            self.in_flight_requests += 1
            self.install_signal_handler()
        else:
            print(f"Skipping request: Service is shutting down.")
            return False

    def send_buffered_data(self):
        while self.buffered_data_count > 0:
            data = get_next_buffered_data()
            if not self.shutdown_requested:
                # Send the buffered data
                pass
            else:
                break

# Example usage:
service = GracefulShutdownService()
service.accept_new_work("Task1")
service.process_request("Request1")
service.send_buffered_data()
```

**Example input / output:**

```text
Input:
- Send a SIGTERM signal to the service

Output:
1. The `handle_signal` method is called.
2. In-flight requests are processed and marked for completion.
3. Buffers are flushed until the shutdown deadline or all data is sent.
4. New work is stopped, and existing in-flight requests complete before shutting down.
```

**Data:**

```json
{
  "in_flight_requests": 0,
  "buffered_data_count": 5,
  "shutdown_requested": false
}
```

**Flow:**

1. No
2. Yes
3. No
4. Receive SIGTERM
5. Is service shutting down?
6. Start graceful shutdown
7. Process and complete in-flightrequests
8. Flush buffers
9. Stop accepting new work
10. Check if hard deadline isreached
11. Force shutdown
12. Shut down service

