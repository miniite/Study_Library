---
type: "practice"
difficulty: "Intermediate"
---

# Your gateway's access log write is synchronous and blocks request threads under high volume; how do you make logging async with a bounded buffer, and what's your policy when the buffer fills during a log-backend outage?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Connection and Resource Limits](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Connection_and_Resource_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To address the issue of synchronous access logs blocking request threads under high volume, we will implement an asynchronous logging mechanism using a bounded buffer. This involves setting up a producer-consumer pattern where requests are logged in the buffer and a separate thread writes to the log backend. We’ll define a policy for handling buffer overflow during log backend outages.

#### Triaging the Question
The core of this question revolves around making the gateway's logging process asynchronous to prevent blocking request threads, especially under high load conditions. The interviewer is probing your understanding of producer-consumer patterns, buffer management, and robust error handling mechanisms. Assumptions include that we already have a working synchronous logging system and are now tasked with optimizing it for better performance.

#### First Principles Solution

Buffer Setup: We will use a bounded buffer (e.g., a fixed-size queue) to hold logs temporarily.
Producer-Consumer Pattern: The request handler acts as the producer, adding log entries to the buffer. A separate thread (consumer) will handle writing these logs to the backend system asynchronously.
Overflow Policy: We need to define what happens if the buffer fills up during a log-backend outage. This could involve retrying writes when the backend is back online or queuing logs for later processing.

Step-by-Step Solution:

Define the Buffer:

JAVA
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

BlockingQueue<String> logBuffer = new ArrayBlockingQueue<>(BUFFER_SIZE);

Producer (Request Handler):

Add logging requests to the buffer.
JAVA
public class LoggingHandler {
    private final BlockingQueue<String> logBuffer;

    public LoggingHandler(BlockingQueue<String> logBuffer) {
        this.logBuffer = logBuffer;
    }

    public void handleRequest(Request request) {
        try {
            // Log relevant information to the buffer
            String logEntry = createLogEntry(request);
            logBuffer.put(logEntry);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("Failed to add log entry", e);
        }
    }

    private String createLogEntry(Request request) {
        // Create a structured log entry
        return "Request ID: " + request.id() + ", Method: " + request.method() + ", Path: " + request.path();
    }
}

#### Cons
umer (Logging Thread):

#### Cons
ume logs from the buffer and write to backend.
JAVA
public class LoggingWriter implements Runnable {
    private final BlockingQueue<String> logBuffer;

    public LoggingWriter(BlockingQueue<String> logBuffer) {
        this.logBuffer = logBuffer;
    }

    @Override
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                String logEntry = logBuffer.take();
                writeLogToBackend(logEntry);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    private void writeLogToBackend(String logEntry) {
        // Code to write the log entry to a backend system
        System.out.println("Writing: " + logEntry);
    }
}

Buffer Management and Policy During Outages:

Implement retry logic for failed writes.
JAVA
private void writeLogToBackend(String logEntry) {
    try {
        // Attempt to write the log entry to a backend system
        if (!writeLogEntry(logEntry)) {
            // If writing fails, queue it for later processing
            logBuffer.put(logEntry);
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}

private boolean writeLogEntry(String logEntry) {
    try {
        // Simulate a backend failure
        if (!isBackendAvailable()) {
            throw new BackendUnreachableException("Backend is unreachable");
        }

        // Write the log entry to a backend system
        return true;
    } catch (BackendUnreachableException e) {
        System.err.println(e.getMessage());
        return false;
    }
}

private boolean isBackendAvailable() {
    // Simulate check for backend availability
    return Math.random() > 0.5; // 50% chance of being available
}

#### Closing Summary
and Common Mistakes/Follow-Ups: Implementing asynchronous logging with a bounded buffer ensures that the gateway’s request handling is not blocked by slow log writing processes, especially under high load. The key steps include setting up a producer-consumer pattern and defining policies for buffer overflow during backend outages.

#### Common mistakes
to avoid:

Not handling buffer overflows properly.
Failing to account for transient failures in the backend.
Overlooking the importance of thread safety and synchronization issues in concurrent access.

#### Follow-ups
might include discussing how to measure and tune performance, ensuring data integrity, and optimizing log buffering strategies based on real-world traffic patterns.

