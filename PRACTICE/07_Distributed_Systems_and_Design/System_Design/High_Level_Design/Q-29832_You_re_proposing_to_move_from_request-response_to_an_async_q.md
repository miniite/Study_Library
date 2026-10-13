---
type: "practice"
difficulty: "Deep"
---

# You're proposing to move from request-response to an async/queued model for a slow workflow — a skeptical PM asks how users still get instant feedback and how you handle failures they can't see; answer them.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fan-out Trade-offs](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In modern software architectures, moving from synchronous request-response models to asynchronous queuing is often necessary to handle slow workflows without blocking user interfaces or server resources. This approach is particularly useful when dealing with I/O-bound tasks such as database queries, file operations, or external service calls that can be time-consuming and unpredictable. The key challenge here is ensuring that users still perceive the system as responsive while handling failures gracefully.

The PM’s concern about instant feedback and hidden failures highlights the importance of designing a robust and transparent user experience. To address these concerns, we need to implement mechanisms for real-time updates and comprehensive error management in our asynchronous model.

#### Triaging the Question
Understanding the Requirements: The PM is skeptical because they expect users to receive immediate responses, but with an async/queued system, this might not be possible. They are also worried about hidden failures that could affect user trust.

#### Assumptions
We have a slow workflow that can be offloaded from the main thread without blocking it.
The primary goal is to improve overall system throughput and responsiveness.
User experience must remain seamless.

#### First-Principles Solution
Real-Time Feedback Mechanism: Implement a push-based notification system where users receive updates in real-time, even if the underlying task takes longer than expected. This can be achieved through web sockets or server-sent events (SSE).
Progress Indicators and Status Updates: Provide clear visual indicators such as progress bars or status messages to inform users about the current state of their request.
Error Handling and Recovery: Design a robust error handling mechanism that ensures all failures are logged, tracked, and communicated back to the user if necessary.
Real-Time Feedback Mechanism

To achieve real-time feedback, we can use web sockets or SSE. Here’s how these technologies work:

Web Sockets: A full-duplex communication channel over a single TCP connection, allowing bi-directional data transfer.
Server-Sent Events (SSE): Allows servers to push events to clients in a stream format.

Both methods enable us to send incremental updates back to the client as the workflow progresses. For example:

JAVASCRIPT
// Example using Server-Sent Events

#### cons
t eventSource = new EventSource('/api/progress');

eventSource.onmessage = function(event) {

#### cons
t status = JSON.parse(event.data);

#### cons
ole.log('Progress: ' + status.progress + '%');
};

eventSource.onerror = function() {

#### cons
ole.error('Could not connect to the server.');
};
Progress Indicators and Status Updates

We can display a progress bar or status messages on the UI to give users an idea of where their request stands. For instance:

Displaying "Processing, please wait..."
Showing a loading spinner while processing data.
Updating the progress bar as tasks complete.
Error Handling and Recovery

For error handling, we need a system that logs all failures and provides meaningful feedback to the user if necessary. This can be achieved through a combination of centralized logging services (e.g., ELK Stack) and custom error messages sent back to the client.

PYTHON
# Example in Python using Flask
@app.route('/api/process')
def process_request():
    try:
        # Perform slow operation
        result = perform_slow_operation()
        return jsonify({'status': 'success', 'result': result})
    except Exception as e:
        logging.error(f"Error processing request: {str(e)}")
        return jsonify({'status': 'failure', 'message': str(e)}), 500

# Example in JavaScript
fetch('/api/process')
  .then(response => response.json())
  .then(data => {
    if (data.status === 'success') {

#### cons
ole.log('Request processed successfully:', data.result);
    } else {
      alert(`Error: ${data.message}`);
    }
  })
  .catch(error => {

#### cons
ole.error('Fetch error:', error);
  });

#### Closing Summary
and Common Mistakes

In conclusion, moving from a synchronous request-response model to an asynchronous queuing system can significantly improve the performance of slow workflows while maintaining user satisfaction. By implementing real-time feedback mechanisms, progress indicators, and robust error handling, we ensure that users receive timely updates and are informed about any issues.

#### Common mistakes
include:

Overlooking real-time communication channels like web sockets or SSE.
Failing to provide clear status updates to the user.
Not logging errors comprehensively or communicating them effectively to the user.

**Code:**

```

// Server-Sent Events example in Node.js const express = require('express'); const app = express(); const http = require('http').createServer(app);

app.get('/api/progress', (req, res) => { let counter = 0; const intervalId = setInterval(() => { counter++; res.write(data: ${counter}%\n\n); if (counter >= 100) { clearInterval(intervalId); res.end(); } }, 500); });

http.listen(3000, () => console.log('Listening on port 3000'));

CODE
Copy

```python
# Flask example for error handling
from flask import jsonify

@app.route('/api/process')
def process_request():
    try:
        result = perform_slow_operation()
        return jsonify({'status': 'success', 'result': result})
    except Exception as e:
        logging.error(f"Error processing request: {str(e)}")
        return jsonify({'status': 'failure', 'message': str(e)}), 500
```

**Data:**

```json
{
  "progress": {
    "current": 30,
    "total": 100
  }
}
```

