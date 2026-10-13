---
type: "practice"
difficulty: "Intermediate"
---

# How would you load the model from S3 or a cloud bucket at Flask startup rather than bundling it in the code repo?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Flask vs Dedicated Model Servers](<../../../../TOOLS/Python/Flask/Flask_vs_Dedicated_Model_Servers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to understand how to integrate Flask with AWS S3 for model loading at runtime. The key is to load the model dynamically when the Flask application starts up, rather than bundling it in the code repository. This approach ensures that our model can be updated without redeploying the entire application.

First, let's break down what we're doing:

Understanding AWS S3: We will use the AWS SDK for Python (Boto3) to interact with S3.
Flask Initialization: We need to ensure that the Flask app loads the model during its initialization process.
Model Loading Logic: Implement a function to download and load the model from S3.

Here's a step-by-step breakdown of how we can achieve this:

Step 1: Set Up AWS Credentials

To interact with AWS, you need to set up your credentials. You can use environment variables or configuration files for this purpose.

PYTHON
import os
AWS_ACCESS_KEY_ID = os.getenv('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.getenv('AWS_SECRET_ACCESS_KEY')
Step 2: Configure Flask App

We'll initialize the Flask app and configure it to load the model from S3 at startup.

PYTHON
from flask import Flask, request
import boto3
import pickle

app = Flask(__name__)

# Define a function to load the model from S3
def load_model_from_s3(model_bucket_name, model_file_key):
    s3_client = boto3.client('s3', aws_access_key_id=AWS_ACCESS_KEY_ID,
                             aws_secret_access_key=AWS_SECRET_ACCESS_KEY)

    obj = s3_client.get_object(Bucket=model_bucket_name, Key=model_file_key)
    model_data = obj['Body'].read()

    # Assuming the model is a pickle file
    with open('model.pkl', 'wb') as f:
        f.write(model_data)

    return pickle.load(open('model.pkl', 'rb'))

# Load the model when Flask app starts
app.before_first_request(load_model_from_s3('my-model-bucket', 'model.pkl'))
Step 3: Define Endpoints and Model Usage

Now that the model is loaded, we can define endpoints to use it.

PYTHON
@app.route('/predict', methods=['POST'])
def predict():
    # Assuming input data comes in JSON format
    data = request.get_json()
    prediction = model.predict([data])
    return {'prediction': prediction.tolist()}
Step 4: Run the Flask App

Finally, run your Flask app.

PYTHON
if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes:

By following these steps, we can dynamically load our model from S3 at startup without hardcoding it in our application. Key points to remember include setting up AWS credentials securely and handling potential errors during the download process.

#### Common mistakes
include not configuring Flask's before_first_request properly or forgetting to secure your AWS credentials.

