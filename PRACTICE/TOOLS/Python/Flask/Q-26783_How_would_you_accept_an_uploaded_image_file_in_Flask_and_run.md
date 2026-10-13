---
type: "practice"
difficulty: "Intermediate"
---

# How would you accept an uploaded image file in Flask and run it through an image-classification model?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Flask vs Dedicated Model Servers](<../../../../TOOLS/Python/Flask/Flask_vs_Dedicated_Model_Servers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle the uploading of an image file in a Flask application, we need to create a simple web form that allows users to select their image files. The Flask backend will then receive these files and process them using an image classification model. Here’s a step-by-step guide on how this can be achieved.

#### Triage

The question is asking for a method to accept uploaded image files in a Flask application, passing these files through an image-classification model. The interviewer might be looking at:

How you handle file uploads in Flask.
Integration with the model inference process.
Error handling and edge cases.
Code structure and best practices.

#### First Principles Solution
Setup Flask Application: Create a basic Flask application to serve as the backend.
HTML Form for File Upload: Design an HTML form that allows users to select and upload image files.
Flask Route to Handle Uploads: Set up a route in Flask to receive the uploaded file from the front end.
Model Integration: Use the uploaded image file as input to your model and process it accordingly.
Error Handling: Ensure robust error handling, such as checking if files are of the correct type or size.

#### Implementation

Let’s start by setting up a simple Flask application with an endpoint for uploading images.

Step 1: Setup Flask Application
PYTHON
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return "No file part", 400

    file = request.files['file']

    # If the user does not select a file, browser also
    # submits an empty part without filename
    if file.filename == '':
        return "No selected file", 400

    if file:
        # Save the file to disk (for simplicity) or process it directly.
        # For this example, we'll just save it and pass to a dummy model.

        # Dummy model: Assume we have a function `predict_image` that takes an image path as input
        from PIL import Image
        import numpy as np

        def predict_image(image_path):
            # Placeholder for actual prediction logic
            return {"prediction": "cat", "confidence": 0.8}

        file_path = f"./uploads/{file.filename}"
        file.save(file_path)

        result = predict_image(file_path)
        return jsonify(result), 200

if __name__ == '__main__':
    app.run(debug=True)
Step 2: HTML Form for File Upload

Create a simple HTML form that users can use to upload images:

HTML
<!DOCTYPE html>
<html>
<head>
    <title>Image Upload</title>
</head>
<body>
    <h1>Upload Image</h1>
    <form action="/upload" method="post" enctype="multipart/form-data">
        Select image file to upload:
        <input type="file" name="file" id="file"><br><br>
        <input type="submit" value="Upload File" name="submit">
    </form>
</body>
</html>

#### Closing Summary

In summary, we have created a basic Flask application with an endpoint to handle image uploads. The form on the frontend allows users to select and submit their images. On the backend, we check for valid file inputs, save them temporarily (or process directly), and then pass the input to our model. This approach ensures that only valid files are processed and includes error handling to manage invalid cases.

#### Common Mistakes/Follow-Ups
Ignoring Edge Cases: Ensure you handle empty uploads or non-image files.
Security Concerns: Validate file types and sizes; avoid saving sensitive information on disk unnecessarily.
Model Performance: Consider the performance implications of running inference in the Flask application versus an external service.

