---
type: "practice"
difficulty: "Intermediate"
---

# How do you write a FastAPI endpoint that accepts an uploaded image and passes it to an image classification model?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ File Uploads and Inference](<../../../../TOOLS/Python/FastAPI/File_Uploads_and_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a FastAPI endpoint that accepts an uploaded image and passes it to an image classification model, we need to follow several steps from first principles. This involves setting up FastAPI with necessary dependencies, defining the API route to accept POST requests containing images, handling file upload, processing the image using an image classification model, and returning results back to the client.

Firstly, let's understand what the question is really asking:

The interviewer wants you to demonstrate your understanding of setting up a FastAPI application.
They are probing your ability to handle file uploads via HTTP requests.
You need to familiarize yourself with integrating external models or services within the FastAPI framework.
There might be an underlying assumption that you have some familiarity with image classification models and their integration in Python.

Now, let's break down the solution step by step:

Set Up Your Environment: Ensure you have Python installed along with FastAPI and Uvicorn (an ASGI server for FastAPI).
Define the API Route: Create a route that will accept POST requests containing an image file.
Handle File Uploads: Use FastAPI's form-data handling capabilities to process the uploaded image.
Process the Image: Pass the processed image data to your image classification model.
Return Results: Send back the results of the classification to the client.
Step 1: Set Up Your Environment

First, ensure you have a virtual environment set up and install FastAPI and Uvicorn:

SH
pip install fastapi uvicorn
Step 2: Define the API Route

Next, define an endpoint in your FastAPI application to accept image files via POST requests.

PYTHON
from fastapi import FastAPI, File, Form, UploadFile
import numpy as np
from PIL import Image
import io

app = FastAPI()

@app.post("/upload_image/")
async def upload_image(file: bytes = File(...)):
    # Process the image and pass it to the model
    result = await process_image(file)
    return {"classification": result}
Step 3: Handle File Uploads

In this example, we use File to accept binary data as a POST request. The bytes parameter indicates that the file is being passed as raw bytes.

Step 4: Process the Image

We need to preprocess the image before passing it to an image classification model. Assuming you have an existing model defined elsewhere (e.g., using TensorFlow or PyTorch), we can use Python's Pillow library for basic image processing.

PYTHON
def process_image(image_bytes):
    # Convert bytes to PIL Image object
    img = Image.open(io.BytesIO(image_bytes))

    # Preprocess the image (resize, normalize, etc.)
    # Example: Resize and convert to numpy array
    img_array = np.array(img.resize((224, 224))) / 255.0

    # Pass the preprocessed image data to your model
    prediction = classify_image(img_array)

    return prediction
Step 5: Return Results

Finally, ensure that you send back a JSON response containing the classification result.

Full Code Example:
PYTHON
from fastapi import FastAPI, File, Form, UploadFile
import numpy as np
from PIL import Image
import io

app = FastAPI()

# Placeholder for model prediction function
def classify_image(image_array):
    # Simulate a model prediction (for example)
    return "Dog"  # This would be the actual classification result from your model

@app.post("/upload_image/")
async def upload_image(file: bytes = File(...)):
    img = Image.open(io.BytesIO(file))
    img_array = np.array(img.resize((224, 224))) / 255.0
    prediction = classify_image(img_array)

    return {"classification": prediction}

#### Closing Summary
+ Common Mistakes/Follow-Ups

This example demonstrates a basic setup for handling image uploads and classifying them using FastAPI. Key points to note include:

Using File to handle binary data.
Preprocessing the image before passing it to the model.
Ensuring proper error handling (e.g., checking if the file is actually an image).

#### Common mistakes
might include not properly handling file types, forgetting to preprocess the image correctly, or using a synchronous model within an asynchronous API.

Would you like me to elaborate on any part of this solution further?

**Code:**

```python
from fastapi import FastAPI, File, Form, UploadFile
import numpy as np
from PIL import Image
import io

app = FastAPI()

# Placeholder for model prediction function
def classify_image(image_array):
    # Simulate a model prediction (for example)
    return "Dog"  # This would be the actual classification result from your model

@app.post("/upload_image/")
async def upload_image(file: bytes = File(...)):
    img = Image.open(io.BytesIO(file))
    img_array = np.array(img.resize((224, 224))) / 255.0
    prediction = classify_image(img_array)

    return {"classification": prediction}
```

**Example input / output:**

```

{ "file": "/9j/4AAQSkZJRgABAQEAAAAAAAD/2wBDAAoHBwcIBwgJCQsJCAcLCgsOCwoODx... }

CODE
Copy
Response:
```json
{
  "classification": "Dog"
}
```

**Flow:**

1. Start
2. Define API Route
3. Handle File Uploads
4. Process Image Data
5. Pass to Model
6. Return Result

