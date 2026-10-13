---
type: "practice"
difficulty: "Intermediate"
---

# How would you offload a slow model inference to a background Celery task and return a job id from a Flask endpoint?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Long-running Work in Flask](<../../../../TOOLS/Python/Flask/Long-running_Work_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To offload a slow model inference to a background Celery task and return a job ID from a Flask endpoint, we need to set up both Flask and Celery in a way that integrates them seamlessly. This process involves several steps: setting up the environment, configuring Celery, creating a Flask app with an endpoint, running the tasks using Celery, and finally returning the job ID to the client.

The core idea is to leverage asynchronous task execution provided by Celery while maintaining a synchronous interface via Flask. Let's break this down step-by-step:

#### Triage
Understanding the Question: The interviewer is probing how you would architect an integration between Flask and Celery such that slow model inference tasks are offloaded to background workers, and the client receives a job ID immediately upon submitting a request.

#### Assumptions
We will use Docker for setup simplicity.
The model inference task is computationally intensive and needs to be run in the background.
Key Steps:
Setup Flask app with an API endpoint.
Configure Celery to handle the background tasks.
Create a route that submits a job and returns a job ID.

#### Solution
Step 1: Setting Up Environment

First, ensure you have Docker installed on your machine. We will use Docker Compose to manage both Flask and Celery services.

BASH
docker-compose up --build

This command will start the services defined in docker-compose.yml.

Step 2: Configuring Flask

Create a basic Flask application that accepts model inference requests and returns job IDs.

app.py

PYTHON
from flask import Flask, jsonify, request
import os

app = Flask(__name__)

@app.route('/inference', methods=['POST'])
def inference():
    data = request.get_json()

    # For now, just return a job ID. In real scenarios, this would trigger Celery task.
    job_id = str(os.urandom(20).hex())
    return jsonify({'job_id': job_id}), 201

if __name__ == '__main__':
    app.run(debug=True)
Step 3: Configuring Celery

Install the necessary dependencies and configure Celery.

requirements.txt

CODE
Flask==2.2.2
Celery==5.2.7
redis==4.3.4

celery_config.py

PYTHON
from celery import Celery

def make_celery(app):
    celery = Celery(
        app.import_name,
        backend=app.config['CELERY_RESULT_BACKEND'],
        broker=app.config['CELERY_BROKER_URL']
    )
    celery.conf.update(app.config)

    class ContextTask(celery.Task):
        def __call__(self, *args, **kwargs):
            with app.app_context():
                return self.run(*args, **kwargs)

    celery.Task = ContextTask
    return celery

app.config.update(
    CELERY_BROKER_URL='redis://localhost:6379',
    CELERY_RESULT_BACKEND='redis://localhost:6379'
)
celery = make_celery(app)

tasks.py

PYTHON
from app import celery
import time
import numpy as np

@celery.task()
def slow_model_inference(input_data):
    # Simulating a long-running task (model inference).
    time.sleep(10)  # Simulate model processing time.

    return {'result': np.random.rand(10)}
Step 4: Running the Flask App with Celery Task

Modify app.py to run tasks asynchronously using Celery.

app.py

PYTHON
from flask import Flask, jsonify, request
import os

app = Flask(__name__)

@app.route('/inference', methods=['POST'])
def inference():
    data = request.get_json()

    # Trigger the Celery task with the provided input.
    result_task = slow_model_inference.delay(data)

    job_id = str(result_task.id)  # Get the job ID from the result task.
    return jsonify({'job_id': job_id}), 201

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary

In summary, we integrated Flask and Celery to handle slow model inference tasks asynchronously. The Flask endpoint accepts a POST request with input data, triggers the task via Celery, and returns a job ID to the client.

#### Common Mistakes
and Follow-Ups
Mistake: Not setting up Redis properly as a message broker.

#### Follow-Up
Discussing scalability and performance implications of using Celery. How could we handle more tasks without compromising performance?

**Code:**

```python
# app.py
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/inference', methods=['POST'])
def inference():
    data = request.get_json()
    result_task = slow_model_inference.delay(data)
    job_id = str(result_task.id)
    return jsonify({'job_id': job_id}), 201

if __name__ == '__main__':
    app.run(debug=True)

# celery_config.py
from celery import Celery

def make_celery(app):
    celery = Celery(
        app.import_name,
        backend=app.config['CELERY_RESULT_BACKEND'],
        broker=app.config['CELERY_BROKER_URL']
    )
    celery.conf.update(app.config)

    class ContextTask(celery.Task):
        def __call__(self, *args, **kwargs):
            with app.app_context():
                return self.run(*args, **kwargs)

    celery.Task = ContextTask
    return celery

app.config.update(
    CELERY_BROKER_URL='redis://localhost:6379',
    CELERY_RESULT_BACKEND='redis://localhost:6379'
)
celery = make_celery(app)

# tasks.py
from app import celery
import time
import numpy as np

@celery.task()
def slow_model_inference(input_data):
    time.sleep(10)  # Simulate model processing time.
    return {'result': np.random.rand(10)}
```

**Example input / output:**

```text
Input:
{
  "data": {"feature_1": 3.4, "feature_2": 2.7}
}

Output:
{
  "job_id": "f698c9e5-7a3f-4fdd-bb0d-d73d0252c7ea"
}
```

**Flow:**

1. Client sends request to Flask
2. Flask creates Celery task
3. Celery worker processes thetask asynchronously
4. Celery returns job ID to Flask
5. Flask returns job ID to Client

