---
type: "practice"
difficulty: "Deep"
---

# How would you enforce a maximum request body size on a REST endpoint, and what status code do you return when exceeded?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ HTTP Status Codes](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Status_Codes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a maximum request body size on a REST endpoint, we need to implement validation logic that checks the incoming payload's size before processing it. This is crucial for maintaining server stability and preventing denial-of-service attacks. The status code typically returned when the request exceeds this limit is 413 Payload Too Large.

Let's break down how to achieve this from first principles.

**Code:**

```

from flask import Flask, request

app = Flask(name)

@app.route('/upload', methods=['POST']) def upload_file(): max_size_kb = 500 # Maximum allowed size in KB max_limit_in_bytes = max_size_kb * 1024

CODE
Copy
if 'file' not in request.files:
    return "No file part", 400

file = request.files['file']

if file.filename == '':
    return "No selected file", 400

# Check the size of the file
if len(file.stream.read()) > max_limit_in_bytes:
    return "File too large", 413

file.stream.seek(0)
# Proceed with processing the file...
CODE
Copy

```python
# For a more robust solution, we can use Flask's limit decorator to set this directly on the route.
from flask import Flask, request

app = Flask(__name__)

@app.route('/upload', methods=['POST'])
def upload_file():
    max_size_kb = 500  # Maximum allowed size in KB
    app.config['MAX_CONTENT_LENGTH'] = max_size_kb * 1024 * 1024  # Convert to bytes

    if request.content_length > max_size_kb * 1024:  # Check directly from the request header
        return "File too large", 413

    file = request.files['file']

    if file.filename == '':
        return "No selected file", 400

    file.stream.seek(0)
    # Proceed with processing the file...
PYTHON
Copy
# Example using FastAPI to enforce max size in a more modern framework
from fastapi import File, UploadFile, FastAPI, HTTPException

app = FastAPI()

@app.post("/uploadfile/")
async def upload_file(file: UploadFile = File(...)):
    max_size_kb = 500  # Maximum allowed size in KB
    max_limit_in_bytes = max_size_kb * 1024 * 1024

    contents = await file.read()

    if len(contents) > max_limit_in_bytes:
        raise HTTPException(status_code=413, detail="File too large")

    # Proceed with processing the file...
PYTHON
Copy
# For a more advanced approach using middleware in Express.js
const express = require('express');
const app = express();

app.use(express.json({ limit: '500kb' }));  // Set max request body size

app.post('/upload', (req, res) => {
    if (req.body.length > 500 * 1024) {  // Check the raw body
        return res.status(413).send("File too large");
    }

    // Proceed with processing the file...
});
```

