---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle file uploads in Flask and validate the uploaded file?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Error Handling](<../../../../TOOLS/Python/Flask/Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with file uploads in Flask, it's essential to focus on both the technical implementation and validation of the uploaded files. This task involves setting up a form that allows users to upload files, handling the file upload server-side using Flask, and implementing robust validation rules to ensure only valid and safe files are processed.

#### Triage

The question is essentially asking how to set up a form for file uploads in Flask, handle the file uploads on the server side, and validate these files. The interviewer might be probing your understanding of Flask's request handling capabilities, form submission techniques, and security considerations around file uploads.

#### Assumptions
We are using HTML forms with enctype="multipart/form-data" to allow file uploads.
Validation is necessary to ensure only specific types of files are uploaded.
The application should handle large files efficiently without compromising performance or security.

#### First Principles Solution

To handle file uploads in Flask, we first need to set up a form for the user to upload a file. We then use Flask's request handling capabilities to process and validate the uploaded file on the server side. Here’s how you can implement this step by step:

Set Up an HTML Form: Create an HTML form that allows users to select files.
Configure Flask: Use Flask's request object to access the uploaded file.
Validate the File: Check if the file is of a valid type, size, and other necessary conditions.

#### Implementation
Step 1: Set Up an HTML Form

Create an HTML form that users can use to upload files:

HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>File Upload</title>
</head>
<body>
    <h1>Upload a File</h1>
    <form action="/upload" method="post" enctype="multipart/form-data">
        <input type="file" name="file" id="file">
        <button type="submit">Upload</button>
    </form>
</body>
</html>
Step 2: Configure Flask to Handle the Form

In your Flask application, handle the form submission and process the uploaded file:

PYTHON
from flask import Flask, request, redirect, url_for

app = Flask(__name__)

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return "No file part", 400

    file = request.files['file']

    # If the user does not select a file, the browser submits an
    # empty file without a filename.
    if file.filename == '':
        return "No selected file", 400

    if file:
        # Validate the file type and size here
        allowed_extensions = {'txt', 'pdf', 'png', 'jpg', 'jpeg'}
        if not '.' in file.filename or file.filename.rsplit('.', 1)[1].lower() not in allowed_extensions:
            return "File type not allowed", 400

        # Check the file size (e.g., maximum of 5MB)
        max_size = 5 * 1024 * 1024
        if file.content_length > max_size:
            return "File too large", 413

        filename = secure_filename(file.filename)
        file.save(os.path.join('uploads', filename))

        return f"File {filename} has been uploaded successfully.", 201

    return "Unknown error occurred during upload", 500

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, handling file uploads in Flask involves setting up a form for users to select files, processing the files on the server side using Flask's request handling capabilities, and implementing validation rules to ensure only specific types of files are accepted. Key areas to consider include checking if the file exists, validating its type, ensuring it is not too large, and securely saving it.

#### Common mistakes
include:

Not checking if a file was actually uploaded.
Allowing any file type without restriction, which can be a security risk.
Failing to secure filenames using secure_filename.

For follow-ups, you might want to discuss handling multiple files at once, implementing asynchronous processing for large files, or integrating with cloud storage services like AWS S3.

**Code:**

```python
from flask import Flask, request, redirect, url_for, secure_filename
import os

app = Flask(__name__)

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return "No file part", 400

    file = request.files['file']

    # If the user does not select a file, the browser submits an
    # empty file without a filename.
    if file.filename == '':
        return "No selected file", 400

    if file:
        # Validate the file type and size here
        allowed_extensions = {'txt', 'pdf', 'png', 'jpg', 'jpeg'}
        if not '.' in file.filename or file.filename.rsplit('.', 1)[1].lower() not in allowed_extensions:
            return "File type not allowed", 400

        # Check the file size (e.g., maximum of 5MB)
        max_size = 5 * 1024 * 1024
        if file.content_length > max_size:
            return "File too large", 413

        filename = secure_filename(file.filename)
        file.save(os.path.join('uploads', filename))

        return f"File {filename} has been uploaded successfully.", 201

    return "Unknown error occurred during upload", 500
```

