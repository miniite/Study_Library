---
type: "practice"
difficulty: "Intermediate"
---

# What does under-fetching look like in an AI app where the client makes many small REST calls, and how could one combined endpoint help?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Under-fetching is a common issue in applications that make frequent, small REST API calls, especially when dealing with large datasets or complex data structures. In an AI application where the client makes many small REST calls to retrieve partial data, under-fetching might occur if the client only requests a part of the required information for each call. This can lead to inefficient use of network resources and increased latency, as well as redundant data processing at both the client and server sides.

To illustrate this, consider an AI application that processes images in batches. The client makes multiple REST calls to fetch different parts of an image dataset. If under-fetching occurs, it might only request a small portion of each image or make many calls for a single image, leading to inefficient data transmission and processing.

A combined endpoint can help mitigate this issue by aggregating the necessary data in one call rather than making multiple small requests. This approach reduces network overhead, improves response time, and ensures that all required information is fetched in a single, efficient operation.

To reason from first principles, let's break down the problem:

Under-fetching Scenario:

Client makes frequent, small REST calls.
Each call fetches only a part of the data needed for processing.

#### Example
Fetching different sections of an image dataset in separate API requests.

Combined Endpoint Solution:

Create a single endpoint that aggregates all necessary data in one request.
This combined endpoint would handle the logic to fetch and process the required information efficiently.

#### Example
A single endpoint could be designed to retrieve multiple sections of an image dataset in a single call, reducing the number of requests.

Let's consider a practical example where we have an AI application that processes images. The client needs to fetch and process different parts of an image for training purposes.

#### Triaging the Question
What is being probed: The interviewer is looking at your understanding of REST API design principles, data fetching strategies, network efficiency, and the ability to optimize client-server interactions.

#### Assumptions
The application processes images in batches.
The client makes multiple small calls for different parts of an image dataset.
Network latency and bandwidth are significant concerns.
Step-by-Step Solution
Identify the Problem: Determine that under-fetching is causing inefficiencies due to frequent, small REST calls.
Design a Combined Endpoint:
Create an endpoint like /images/fetch which accepts parameters such as image ID and section IDs.
The server-side logic processes these requests and combines the necessary sections into one response.
Optimize Client-Side Logic:
Ensure the client sends only the required data for processing in a single call.
Implement caching mechanisms if parts of the dataset are frequently requested.

#### Example
Implementation
Combined Endpoint (Python Flask)
PYTHON
from flask import Flask, request, jsonify

app = Flask(__name__)

# Mock image storage and sections
image_data_store = {
    "1": ["section1.png", "section2.png", "section3.png"],
    # ... more images
}

@app.route('/images/fetch', methods=['GET'])
def fetch_image_sections():
    try:
        image_id = request.args.get('image_id')
        section_ids = request.args.getlist('section_id')

        if not image_id or not section_ids:
            return jsonify({"error": "Missing required parameters"}), 400

        # Fetch necessary sections
        fetched_sections = [image_data_store[image_id][int(id) - 1] for id in section_ids]
        response = {"sections": fetched_sections}

        return jsonify(response)
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)
Sample Data
JSON
{
  "image_id": "1",
  "section_ids": ["1", "2"]
}

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Not considering edge cases, such as invalid image IDs or section IDs.
Failing to handle large datasets efficiently.
Overlooking client-side optimization.

#### Follow-Ups

How would you further optimize this combined endpoint?
What other strategies could be used to reduce network overhead in an AI application?

**Code:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

image_data_store = {
    "1": ["section1.png", "section2.png", "section3.png"],
}

@app.route('/images/fetch', methods=['GET'])
def fetch_image_sections():
    image_id = request.args.get('image_id')
    section_ids = request.args.getlist('section_id')

    if not image_id or not section_ids:
        return jsonify({"error": "Missing required parameters"}), 400

    fetched_sections = [image_data_store[image_id][int(id) - 1] for id in section_ids]
    response = {"sections": fetched_sections}

    return jsonify(response)

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```text
Input:
GET /images/fetch?image_id=1&section_id=1&section_id=2

Output:
{
  "sections": ["section1.png", "section2.png"]
}
```

**Data:**

```json
{
  "image_id": "1",
  "section_ids": ["1", "2"]
}
```

**Flow:**

1. Yes
2. No
3. Client Request
4. Is Combined EndpointAvailable?
5. Fetch Data in One Call
6. Make Multiple Small Calls
7. Process Efficiently
8. Increase Latency, NetworkOverhead

