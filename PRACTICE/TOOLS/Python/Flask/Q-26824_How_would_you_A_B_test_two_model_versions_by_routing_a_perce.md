---
type: "practice"
difficulty: "Deep"
---

# How would you A/B test two model versions by routing a percentage of Flask traffic to each?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Routing and Blueprints](<../../../../TOOLS/Python/Flask/Routing_and_Blueprints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To A/B test two model versions using Flask, we need to route a certain percentage of incoming requests to one version while the remaining requests are directed to the other. The key concept here is routing based on probabilistic logic rather than deterministic rules. This approach allows us to maintain a fair comparison between the two models without skewing results due to fixed ratios or biases.

#### Triage

The core question is how to implement A/B testing in Flask by dynamically routing traffic to different model versions. The interviewer might be probing your understanding of Flask routing, probabilistic logic, and possibly caching mechanisms to ensure fair testing. Assumptions include that both models are ready for deployment, the Flask application is set up, and we need to manage state across requests if necessary.

#### Solution
from First Principles
Define the Traffic Distribution: Decide on a percentage of traffic (e.g., 50%) that should go to each model version.
Implement Random Routing Logic: Use Python’s random module or similar logic to decide which model version to serve based on the predefined distribution.
Cache Results if Necessary: To avoid recalculating for the same user, use caching mechanisms (like Flask’s session or Redis).
Track Results: Maintain logs or metrics to monitor performance and results of both models.

Let's walk through these steps in detail:

Step 1: Define Traffic Distribution

We decide that 50% of traffic should go to Model Version A, while the other 50% goes to Model Version B. This can be adjusted as needed based on specific testing goals.

Step 2: Implement Random Routing Logic

Here’s how we can implement this in Flask:

PYTHON
from flask import Flask, request, session
import random

app = Flask(__name__)

# Set a secret key for sessions (not strictly necessary but good practice)
app.secret_key = 'your_secret_key'

@app.route('/predict')
def predict():
    # Generate a random number between 0 and 100
    rand_num = random.randint(0, 99)

    if rand_num < 50:
        model_version = "A"
    else:
        model_version = "B"

    # Log or cache the decision (optional but recommended)
    session['model_version'] = model_version

    # Return the prediction from the chosen model version
    return f"Model Version {model_version} was used for prediction."

# Optional: Add more routes and logic as needed
Step 3: Cache Results if Necessary

To avoid serving different models to the same user, we can use Flask’s session object or a caching layer like Redis:

PYTHON
from flask import Flask, request, session

app = Flask(__name__)
app.secret_key = 'your_secret_key'

@app.route('/predict')
def predict():
    # Check if model version is already cached for this session
    if 'model_version' not in session:
        rand_num = random.randint(0, 99)
        if rand_num < 50:
            model_version = "A"
        else:
            model_version = "B"

        # Cache the decision
        session['model_version'] = model_version

    return f"Model Version {session['model_version']} was used for prediction."
Step 4: Track Results

To track results, we can use Flask’s logging capabilities or an external service like Google Analytics:

PYTHON
import logging

app.logger.setLevel(logging.INFO)

@app.route('/predict')
def predict():
    rand_num = random.randint(0, 99)
    if rand_num < 50:
        model_version = "A"
    else:
        model_version = "B"

    session['model_version'] = model_version
    app.logger.info(f"User {request.remote_addr} was served Model Version {model_version}")

    return f"Model Version {model_version} was used for prediction."

#### Closing Summary
and Common Mistakes/Follow-Ups

By following the above steps, we can effectively A/B test two model versions using Flask. Key takeaways include defining clear traffic distribution, implementing random routing logic, considering caching to maintain consistency, and tracking results. Some common mistakes might include forgetting to cache decisions for users or not properly logging results.

For follow-ups, you could discuss more advanced techniques like using load balancers for A/B testing at a higher level, handling edge cases such as sudden spikes in traffic, and ensuring the Flask app is secure.

**Comparison:**

```
User IP	Route	Response	Log Message
192.168.0.1	/predict	Model Version A	User 192.168.0.1 was served Model Version A
192.168.0.2	/predict	Model Version B	User 192.168.0.2 was served Model Version B
```

**Code:**

```

from flask import Flask, request, session import random

app = Flask(name)

Set a secret key for sessions (not strictly necessary but good practice)

app.secret_key = 'your_secret_key'

@app.route('/predict') def predict(): # Generate a random number between 0 and 100 rand_num = random.randint(0, 99)

CODE
Copy
if rand_num < 50:
    model_version = "A"
else:
    model_version = "B"

# Log or cache the decision (optional but recommended)
session['model_version'] = model_version

# Return the prediction from the chosen model version
return f"Model Version {model_version} was used for prediction."
Optional: Add more routes and logic as needed
CODE
Copy

```python
from flask import Flask, request, session

app = Flask(__name__)
app.secret_key = 'your_secret_key'

@app.route('/predict')
def predict():
    # Check if model version is already cached for this session
    if 'model_version' not in session:
        rand_num = random.randint(0, 99)
        if rand_num < 50:
            model_version = "A"
        else:
            model_version = "B"

        # Cache the decision
        session['model_version'] = model_version

    return f"Model Version {session['model_version']} was used for prediction."
PYTHON
Copy
import logging

app.logger.setLevel(logging.INFO)

@app.route('/predict')
def predict():
    rand_num = random.randint(0, 99)
    if rand_num < 50:
        model_version = "A"
    else:
        model_version = "B"

    session['model_version'] = model_version
    app.logger.info(f"User {request.remote_addr} was served Model Version {model_version}")

    return f"Model Version {model_version} was used for prediction."
```

**Example input / output:**

```text
- **Input**: `GET /predict` with a user IP address.
- **Output**:
  - For a random number less than 50 (say, 47): `"Model Version A"`.
  - For a random number greater than or equal to 50 (say, 63): `"Model Version B"`.

This setup ensures that the model version is served based on a random distribution and logs are created for tracking.
```

**Data:**

```json
{
  "user_ip": "192.168.0.1",
  "route": "/predict",
  "response": {
    "model_version": "A"
  },
  "log_message": "User 192.168.0.1 was served Model Version A"
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Random Number < 50?
5. Model Version A
6. Model Version B
7. Log and Return Model Version

