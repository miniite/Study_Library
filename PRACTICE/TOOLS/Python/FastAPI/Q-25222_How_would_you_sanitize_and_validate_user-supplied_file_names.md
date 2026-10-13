---
type: "practice"
difficulty: "Intermediate"
---

# How would you sanitize and validate user-supplied file names and content types on an upload endpoint in FastAPI?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Custom Responses](<../../../../TOOLS/Python/FastAPI/Custom_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to create a FastAPI endpoint that handles file uploads while ensuring both the filename and content type are sanitized and validated. The goal is to prevent malicious activities like directory traversal or uploading dangerous files.

#### Triage

The core of the question revolves around understanding how to properly sanitize and validate user-supplied filenames and content types in an upload endpoint using FastAPI. We need to ensure that:

The filename doesn't contain any potentially harmful characters.
The file type is allowed (e.g., .png, .jpg).
The request payload adheres to expected constraints.

#### Assumptions
The application will support common image formats, such as .png, .jpg.
We need to handle both form-data and raw multipart requests.
The endpoint should return appropriate error messages for invalid inputs.

#### Solution
(From First Principles)
Define the FastAPI Endpoint: Use FastAPI to define an upload endpoint that accepts a file via form-data or raw multipart request.
Sanitize Filename: Ensure the filename is safe by removing any potentially harmful characters and checking its length.
Validate Content Type: Check if the content type matches one of our allowed types (e.g., image/png, image/jpeg).
Handle Errors Gracefully: Provide meaningful error responses for invalid filenames or unsupported content types.

Let's walk through the implementation step by step:

Step 1: Define the FastAPI Endpoint

We start by importing necessary modules and defining a function to handle file uploads.

PYTHON
from fastapi import FastAPI, UploadFile, HTTPException, File, Form
import os

app = FastAPI()

ALLOWED_EXTENSIONS = {'png', 'jpg'}
Step 2: Sanitize the Filename

We create a helper function sanitize_filename to clean up and validate filenames.

PYTHON
def sanitize_filename(filename: str) -> str:
    # Remove potentially harmful characters
    safe_filename = ''.join([c if c.isalnum() or c in ('.', '_') else '_' for c in filename])
    # Check length constraints
    max_length = 255
    if len(safe_filename) > max_length:
        raise HTTPException(status_code=400, detail="Filename too long")
    return safe_filename

@app.post("/upload/")
async def upload_file(file: UploadFile = File(...)):
    # Sanitize the filename
    sanitized_name = sanitize_filename(file.filename)

    # Further processing and validation
Step 3: Validate Content Type

We use the file.content_type attribute to check if the file type is allowed.

PYTHON
    content_type = file.content_type
    if content_type.split('/')[0] not in ('image'):
        raise HTTPException(status_code=415, detail="Unsupported media type")

    # Check against specific extensions if needed
    ext = os.path.splitext(file.filename)[1].lower()
    if ext[1:] not in ALLOWED_EXTENSIONS:
        raise HTTPException(status_code=400, detail=f"File extension {ext} is not allowed")

    # Save the file to a secure location
Step 4: Handle Errors Gracefully

We ensure that any validation errors are handled by raising appropriate HTTP exceptions.

PYTHON
@app.exception_handler(HTTPException)
async def http_exception_handler(request, exc):
    return {"detail": exc.detail}

@app.get("/")
async def read_root():
    return {"Hello": "World"}

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that uploaded files are both sanitized and validated according to our specified criteria. By using FastAPI's built-in validation features, we can easily handle common edge cases like unsupported file types or filenames with harmful characters.

#### Common Mistakes
Insufficient Sanitization: Not thoroughly cleaning the filename can lead to injection attacks.
Overly Broad Validation: Allowing too many content types without proper checks might introduce security risks.
Error Handling: Failing to provide meaningful error messages can make debugging difficult.

#### Follow-Ups
How would you handle large file uploads (e.g., 100MB+ files)?
What measures would you take to ensure the storage location of these files is secure?
Could you integrate a logging mechanism to track upload activities?

