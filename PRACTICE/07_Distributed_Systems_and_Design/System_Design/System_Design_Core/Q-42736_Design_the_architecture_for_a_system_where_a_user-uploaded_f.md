---
type: "practice"
difficulty: "Deep"
---

# Design the architecture for a system where a user-uploaded file is scanned, transformed, and made available across web and mobile, and the frontend must show accurate progress and a preview the moment it's safe, without exposing an unscanned or half-processed file. Where are the state transitions authoritative?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Frontend Graceful Degradation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The architecture involves creating a robust system that ensures user-uploaded files are processed correctly and securely before being made available to users via web and mobile frontends. The key challenges include ensuring data integrity, maintaining state consistency, and providing real-time progress updates without exposing incomplete or unprocessed files.

#### Triaging the Question

This question is probing your understanding of how to design a system that handles file uploads efficiently while maintaining strict control over data processing and state transitions. Specifically, it’s asking how you manage the flow from upload to final availability, ensuring that only fully processed and safe files are shown to users. The interviewer might be interested in how you handle concurrency, data integrity, and the mechanisms for signaling progress updates.

Key Assumptions
We need a scalable system capable of handling large file uploads.
Users should see accurate real-time progress on both web and mobile devices.
There’s no possibility of showing an unprocessed or half-processed file to users.
The system must handle various types of files, including images, documents, and videos.

#### Solution
from First Principles
1. File Upload
Users upload files via a web form or mobile app.
Files are stored in a temporary storage location (e.g., AWS S3).
A unique identifier is generated for each file upon upload to ensure uniqueness and track progress.
2. File Processing Pipeline
The uploaded file is then sent through a processing pipeline:
Validation: Check file type, size, etc.
Scanning/Transformation: Use appropriate tools or services (e.g., OCR for documents, image analysis for photos) to process the content.
Safety Checks: Apply any necessary safety checks (e.g., malware detection).
Once processed and verified as safe, the file is moved to a production storage location.
3. State Transitions
The state transitions are managed using a central state service or database:
Uploaded: File has been uploaded but not yet processed.
Processing: File is currently being processed.
Safe: File has been fully processed and verified as safe.
Error: Processing failed for some reason.
4. Progress Tracking
A real-time progress tracker updates the state transitions in near-real time:
For each step of the processing pipeline, an update is sent to a central state service.
This ensures that both frontends can accurately reflect the current state of file processing.
Users receive updates through web sockets or long-polling requests.
5. Frontend Interactions
Web and mobile apps interact with the central state service via APIs to fetch the latest state of a file.
If the state is Safe, the frontend can display the preview; otherwise, it shows a loading indicator or error message if in an Error state.
6. Security Measures
Ensure that only fully processed and safe files are served to users:
Use proper access controls on storage services.
Implement rate limiting and other security measures to prevent abuse.
Employ HTTPS for all communications between the client and server.

#### Closing Summary

The architecture ensures a seamless user experience by managing file uploads, processing, and state transitions carefully. The use of a central state service and real-time updates guarantees that users always see accurate progress and previews without being exposed to unprocessed or unsafe content.

#### Common Mistakes
and Follow-Ups
Mistake: Ignoring the need for proper validation and safety checks before making files available.

#### Follow-Up
Discuss how to handle file types not supported by your system, fallbacks in case of processing errors, and ensuring data integrity during transmission.

**Comparison:**

```
State	Description
Uploaded	File has been uploaded but not yet processed.
Processing	File is currently being processed.
Safe	File has been fully processed and verified as safe.
Error	Processing failed for some reason.
```

**Code:**

```python
# Example State Transition Update API
from flask import Flask, request

app = Flask(__name__)

state_db = {}

@app.route('/update_state', methods=['POST'])
def update_state():
    file_id = request.json['file_id']
    new_state = request.json['new_state']

    if file_id in state_db:
        state_db[file_id] = new_state
        return f"State updated for {file_id} to {new_state}"
    else:
        return "File not found", 404

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```text
POST /update_state HTTP/1.1
Host: localhost:5000
Content-Type: application/json

{
  "file_id": "12345",
  "new_state": "Safe"
}

Response:
200 OK
State updated for 12345 to Safe
```

**Data:**

```json
{
  "file_id": "12345",
  "state": "Safe"
}
```

**Flow:**

1. Yes
2. No
3. File Upload
4. Validation
5. Scanning/Transformation
6. Safety Checks
7. Safe?
8. Store in Production
9. Error Handling
10. State Update

