---
type: "practice"
difficulty: "Deep"
---

# How would you design deploy verification so 'the detector is deployed' means 'it scored a known synthetic anomaly and paged the test channel', not just 'the pod is running'?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deployment Verification](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Deployment_Verification.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that "the detector is deployed" means it has scored a known synthetic anomaly and paged the test channel, rather than merely indicating that a pod is running, we need to design a robust CI/CD pipeline with multiple verification stages. This involves setting up automated tests that simulate anomalies, triggering alerts, and verifying if these alerts are correctly routed and handled by our monitoring system.

#### Triaging the Question

The core of this question revolves around ensuring reliability in software deployment through comprehensive testing strategies. The key aspects the interviewer is probing include:

Deployment Verification: Ensuring that the deployment not only succeeds but also performs as intended.
Test Simulations: Creating synthetic anomalies to test the alerting and paging mechanisms.
Integration Testing: Verifying end-to-end workflows from deployment to alert handling.

#### Assumptions
The CI/CD pipeline is already set up for deploying the detector.
There are existing synthetic anomaly generators available or can be developed.
A testing channel exists that can receive and process alerts.

#### First-Principles Solution
Step 1: Define the Deployment Criteria

To make sure "the deployment" implies a successful score on known synthetic anomalies, we need to define clear criteria for success. This includes:

Pod Running: Verify that the pod is up and running.
Anomaly Detection: Ensure the detector has successfully detected a predefined synthetic anomaly.
Step 2: Develop Synthetic Anomalies

Synthetic anomalies should be designed based on typical failure modes or edge cases of the system. For example, if the detector processes time series data, we might generate:

A sudden spike in values.
Abrupt changes in mean and variance.
Periodic oscillations.
Step 3: Implement Automated Testing

We will create automated tests that deploy the detector and then inject synthetic anomalies to see if they trigger alerts. The tests should cover various scenarios including different types of anomalies, varying severities, and edge cases.

Step 4: Set Up Alert Handling Tests

Ensure that when an alert is triggered, it is correctly routed to the test channel for verification. This involves setting up a testing infrastructure where:

Alerts are sent to the test channel.
The test channel processes these alerts.
Verifies if the correct actions are taken (e.g., logging, notifications).
Step 5: Continuous Monitoring and Feedback Loops

Implement continuous monitoring to automatically trigger re-deployment or failover in case of unexpected behavior. Use logging and metrics to track performance and adjust the testing strategy as needed.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, we need a robust verification process that goes beyond mere deployment status checks. This includes setting up synthetic anomalies, automated tests, alert handling mechanisms, and continuous monitoring to ensure reliability and accuracy in our deployments. Common mistakes include:

Overlooking the importance of synthetic testing.
Not having a clear definition of what "success" means for each stage of deployment.
Failing to integrate end-to-end verification from start to finish.

During the interview, it might be useful to discuss potential trade-offs between complexity and reliability, as well as how this solution scales with increasing numbers of detectors or more complex anomaly patterns.

**Comparison:**

```
Anomaly Type	Description	Test Cases
Spike	Sudden change in value	Large spike, Small spike
Oscillation	Repeated fluctuations	High frequency, Low frequency
Step	Action	Expected Result	Actual Result
1	Deploy Detector	Pod is running	Pod Running
2	Inject Spike	Anomaly detected in logs	Anomaly detected
3	Verify Alert	Test channel receives alert	Alert received
```

**Code:**

```python
# Example code for deploying a detector and injecting synthetic anomalies
import subprocess
from time import sleep

def deploy_detector():
    result = subprocess.run(["kubectl", "apply", "-f", "detector.yaml"])
    if result.returncode != 0:
        raise Exception("Detector deployment failed")

def inject_anomaly(anomaly_type):
    if anomaly_type == "spike":
        # Code to generate a spike in data
        pass
    elif anomaly_type == "oscillation":
        # Code to simulate oscillation
        pass

def test_detector():
    deploy_detector()
    sleep(10)  # Wait for the detector to start up
    inject_anomaly("spike")
    sleep(5)  # Allow time for detection and alerting
    # Check if the alert was correctly sent to the test channel
```

**Example input / output:**

```text
Input:
1. Deploy detector with kubectl apply -f detector.yaml
2. Inject synthetic spike anomaly using the Python script

Output:
- The pod starts up and runs successfully.
- A spike in data is generated, triggering an alert.
- The test channel receives a notification indicating that the detector has identified a synthetic spike.

Dry-run:
1. Deploy_detector() -> Pod starts running (Status: Running)
2. inject_anomaly("spike") -> Spike injected into the system
3. sleep(5) -> Wait for detection and alerting to occur
4. Check if the test channel logs contain "Anomaly detected" message.
```

**Data:**

```yaml
# Example YAML file for deploying a detector
apiVersion: apps/v1
kind: Deployment
metadata:
  name: anomaly-detector
spec:
  replicas: 1
  selector:
    matchLabels:
      app: anomaly-detector
  template:
    metadata:
      labels:
        app: anomaly-detector
    spec:
      containers:
      - name: anomaly-detector
        image: <detector-image>
```

**Flow:**

1. Yes
2. Deploy Detector
3. Pod Running?
4. Inject Synthetic Anomaly
5. Detect and Log Anomaly
6. Alert Generated
7. Test Channel Receives Alert
8. Verify Correct Handling

