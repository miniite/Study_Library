---
type: "practice"
difficulty: "Intermediate"
---

# How would you return the scored CSV back to the client as a downloadable file from a FastAPI endpoint?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Custom Responses](<../../../../TOOLS/Python/FastAPI/Custom_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To return a scored CSV as a downloadable file from a FastAPI endpoint, we need to ensure that the HTTP response headers are correctly set to indicate that the content is a downloadable file with the appropriate MIME type and filename. Let's break down the process step by step.

#### Triage

The question is asking how to serve a CSV file generated via a FastAPI endpoint so that it can be downloaded directly from the client’s browser. The core of the problem involves setting up an HTTP response correctly using FastAPI, which requires understanding the basics of FastAPI and HTTP responses.

#### Assumptions
We assume you have already set up your environment with FastAPI.
You have a function or endpoint that generates a CSV file based on some input data.
The client is making a GET request to this endpoint.
Step-by-Step Solution

Generate the CSV File: First, we need to generate the CSV file using Python’s csv module or any other library suitable for your use case. For simplicity, let's assume you have already generated the CSV content and stored it in a variable called csv_content.

Set Up FastAPI Endpoint: Create an endpoint that returns the CSV file as a downloadable attachment.

Here is how you can implement this:

PYTHON
from fastapi import FastAPI, File, UploadFile
import csv

app = FastAPI()

# Function to generate CSV content (example)
def generate_csv(data):
    with open("output.csv", mode="w", newline="", encoding="utf-8") as file:
        writer = csv.writer(file)
        for row in data:
            writer.writerow(row)
    return "output.csv"

@app.get("/download-csv")
async def download_csv():
    # Example data to generate CSV
    data = [
        ["Name", "Age"],
        ["Alice", 20],
        ["Bob", 30]
    ]

    csv_path = generate_csv(data)

    with open(csv_path, "rb") as file:
        content = file.read()

    return FileResponse(
        path=csv_path,
        filename="output.csv",
        media_type="application/octet-stream"
    )
Explanation
CSV Generation: We use the csv module to write data into a CSV file. In this example, we have hardcoded some sample data.
Endpoint Definition: The /download-csv endpoint is defined using FastAPI's @app.get decorator.
File Response: FileResponse from FastAPI is used to return the file as an HTTP response. We pass the path of the CSV file, its filename, and the MIME type (application/octet-stream) which tells the browser that it should treat the content as a binary file (like a downloadable file).

#### Closing Summary

By setting up a FastAPI endpoint with FileResponse, we can return a generated CSV file directly to the client. This approach ensures that the file is correctly marked for download, and the user will be prompted to save or open it in their browser.

#### Common Mistakes/Follow-Ups
Forgetting to Set Headers: Ensure you set the correct Content-Disposition header with attachment; filename="output.csv" to trigger a download.
Handling Large Files: If dealing with large files, consider streaming the file instead of loading it entirely into memory. FastAPI supports streaming using generators or asynchronous code.
Error Handling: Implement proper error handling in case the file generation fails.

