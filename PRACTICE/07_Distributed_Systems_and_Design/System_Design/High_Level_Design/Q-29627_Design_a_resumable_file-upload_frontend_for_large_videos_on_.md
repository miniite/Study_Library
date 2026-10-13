---
type: "practice"
difficulty: "Intermediate"
---

# Design a resumable file-upload frontend for large videos on flaky mobile networks — chunking, retry, progress, and pause/resume across app restarts; where do you keep upload state?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To design a resumable file-upload frontend for large videos on flaky mobile networks, we need to implement chunking and manage states across app restarts. The core idea is to break the video into smaller chunks, upload them in parallel, and use persistent storage to track upload progress and handle retries.

#### Triaging the Question
This question tests your understanding of handling large files over unreliable network connections and state management—critical for a robust user experience. We need to ensure that even if the app is restarted, the upload can continue from where it left off without losing any data or progress. The key aspects are:

Chunking: Breaking down the file into smaller chunks to manage upload tasks.
State Management: Persistently storing and retrieving upload states across app restarts.
Retry Logic: Implementing mechanisms for retries in case of failures.
Progress Reporting: Providing users with a clear understanding of upload progress.

#### First-Principles Solution

Chunking Strategy:
The video file is split into fixed-size chunks (e.g., 5MB each). This reduces the likelihood of a single chunk failing due to network instability.
State Management:
Use local storage (like IndexedDB or AsyncStorage) to store metadata about the upload state, including which chunks have been uploaded and their states (uploaded, failed, etc.).
Retry Logic:
Introduce exponential backoff for retries when a chunk fails to upload due to network issues.
Progress Reporting:
Update the UI with real-time progress indicators based on the number of successfully uploaded chunks.

Let's break down each component in more detail:

Chunking Strategy:

#### Implementation
Use JavaScript's Array.prototype.slice() or a similar method to split the file into fixed-size chunks.
JAVASCRIPT

#### cons
t chunkSize = 5 * 1024 * 1024; // 5MB

#### cons
t videoFile = await fetch(fileInput.files[0]).then(res => res.arrayBuffer());

#### cons
t videoChunks = [];
for (let i = 0; i < videoFile.byteLength; i += chunkSize) {
    videoChunks.push(videoFile.slice(i, Math.min(i + chunkSize, videoFile.byteLength)));
}

State Management:

Storage Mechanism: Use AsyncStorage from React Native for local storage.
JAVASCRIPT

#### cons
t state = await AsyncStorage.getItem('upload_state');
if (state) {
    uploadState = JSON.parse(state);
} else {
    uploadState = {};
}
Persisting State: Save the upload state after each chunk is uploaded or when the app closes.
JAVASCRIPT
await AsyncStorage.setItem('upload_state', JSON.stringify(uploadState));

Retry Logic:

Implement a retry mechanism using exponential backoff for network failures.
JAVASCRIPT

#### cons
t retryUpload = async (chunkIndex) => {
    for (let attempt = 1; attempt <= MAX_RETRIES; attempt++) {
        try {
            await uploadChunk(chunkIndex);
            return;
        } catch (error) {

#### cons
ole.error(`Attempt ${attempt} failed:`, error);
            await sleep(attempt * RETRY_DELAY_MS);
        }
    }
};

Progress Reporting:

Update the UI with a progress bar and detailed status messages.
JAVASCRIPT

#### cons
t progressBar = document.getElementById('progress-bar');

#### cons
t totalChunks = Object.keys(uploadState).length;

#### cons
t completedChunks = Object.keys(uploadState).filter(key => uploadState[key].status === 'success').length;
progressBar.value = (completedChunks / totalChunks) * 100;

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Summary: The design involves breaking the video into chunks, managing state locally to ensure resumability, implementing retry logic for robustness, and updating the UI with progress.

#### Common Mistakes
Overlooking edge cases like network timeouts or errors during chunking.
Failing to properly handle state persistence and retrieval.
Not providing clear user feedback on upload status.

For follow-ups, you might be asked about performance optimization strategies (like parallel uploads), security considerations, or how to integrate this with backend services for large-scale deployments. Be prepared to discuss trade-offs between complexity and reliability in such designs.

**Comparison:**

| Chunk Index | Status   | Bytes Sent |
|-------------|----------|------------|
| 1           | Success  | 5242880    |
| 2           | Failed   | N/A        |
| 3           | Pending  | 262144     |

**Code:**

```javascript
const chunkSize = 5 * 1024 * 1024; // 5MB

const videoFile = await fetch(fileInput.files[0]).then(res => res.arrayBuffer());
const videoChunks = [];
for (let i = 0; i < videoFile.byteLength; i += chunkSize) {
    videoChunks.push(videoFile.slice(i, Math.min(i + chunkSize, videoFile.byteLength)));
}

// Retry mechanism
async function retryUpload(chunkIndex) {
    for (let attempt = 1; attempt <= MAX_RETRIES; attempt++) {
        try {
            await uploadChunk(chunkIndex);
            return;
        } catch (error) {
            console.error(`Attempt ${attempt} failed:`, error);
            await sleep(attempt * RETRY_DELAY_MS);
        }
    }
}
```

**Example input / output:**

| Input (file: large-video.mp4) | Output (upload_state.json) |
|---|---|
| Split file into chunks of 5MB each. State is saved in local storage. | Upload state persisted and retrieved after app restarts, ensuring resumable uploads. Retry logic implemented for chunk failures. Progress tracked with UI updates. |

**Data:**

```json
{
  "upload_state": {
    "chunk1": { "status": "success", "bytes_sent": 5242880 },
    "chunk2": { "status": "failed" },
    "chunk3": { "status": "pending", "bytes_sent": 262144 }
  }
}
```

**Flow:**

1. Success
2. Failed
3. Yes
4. No
5. User selects file
6. Split file into chunks
7. Upload chunk?
8. Update upload state andprogress UI
9. Retry logic (exponentialbackoff)
10. Retry successful?
11. Save partial upload state andexit
12. Restart application, resumefrom saved state

