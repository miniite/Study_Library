---
type: "practice"
difficulty: "Basic"
---

# What is model serving and how is it different from training when you think about the cloud resources each needs?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Multi-cloud Strategy](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/Multi-cloud_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Model serving involves deploying a trained machine learning (ML) model to an environment where it can make real-time predictions or inferences on new data. This process is distinct from training, which focuses on improving a model's accuracy by optimizing its parameters using historical data. In the cloud context, both processes require different types of resources and considerations.

To break down the core idea: during training, you need powerful compute resources to handle the computational demands of optimizing the model's performance over time. Model serving, however, requires more scalable and performant infrastructure that can handle a high volume of inference requests efficiently. Let's delve into each aspect from first principles.

#### Triaging the Question

The question is asking us to differentiate between training and serving in terms of cloud resource requirements. We need to consider the primary goals and resources needed for each process:

Training: The goal is to optimize model parameters by minimizing loss functions using large datasets.
Serving: The goal is to deploy a trained model to make real-time predictions on new data.

We'll assume that we have access to cloud services like AWS, Azure, or Google Cloud. These platforms offer various instance types and managed services tailored for training and serving.

Step-by-Step Solution

Training:

Compute Resources: High-performance GPUs or TPUs are often used to speed up the training process.
Storage: Large storage resources for storing datasets, logs, and checkpoints.
Distributed Computing: Training large models may require distributed computing setups like Apache Spark or TensorFlow's own distributed support.

Serving:

Compute Resources: Lower latency and higher throughput are critical. Cloud providers often offer dedicated inference instances optimized for speed.
Scalability: The ability to scale the number of instances up or down based on demand.
High Availability: Ensuring that serving endpoints can handle a high volume of requests without downtime.

Cloud Resources:

Training: Typically involves EC2 instances with appropriate GPU/TPU types, EBS for storage, and potentially Spot Instances to reduce costs.
Serving: Uses managed services like AWS Sagemaker Inference, Google AI Platform Predictions, or Azure ML Inference Services. These services handle scaling and maintain high availability.

#### Closing Summary

In summary, while training focuses on optimizing a model's performance using powerful compute resources, serving emphasizes deploying the model for real-time predictions with scalable, performant infrastructure. The cloud resources required are different: high-performance GPUs for training versus optimized inference instances for serving.

#### Common Mistakes
and Follow-Ups
Mistake: Confusing the resource requirements between training and serving.

#### Follow-Up
Discussing edge cases such as hybrid models where both training and serving need to be optimized, or discussing cost considerations in scaling deployments.

**Comparison:**

|Resource Type | Training | Serving |
|--------------|----------|---------|
|Compute       | High     | Lower   |
|Storage       | Large    | Moderate|
|Availability  | N/A      | High    |
|Cost          | Costly   | Economical|

**Code:**

```python
# Example of a simple training script using PyTorch
import torch
from torchvision import datasets, transforms

transform = transforms.Compose([transforms.ToTensor()])
dataset = datasets.MNIST(root='./data', train=True, download=True, transform=transform)
dataloader = torch.utils.data.DataLoader(dataset, batch_size=64)

model = torch.nn.Sequential(torch.nn.Linear(28*28, 10), torch.nn.Softmax())
criterion = torch.nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.01)

for epoch in range(5):  # loop over the dataset multiple times
    for i, data in enumerate(dataloader, 0):
        inputs, labels = data

        optimizer.zero_grad()

        outputs = model(inputs.view(-1, 28*28))
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()

# Example of a serving script using Flask
from flask import Flask, request, jsonify
import torch

app = Flask(__name__)

model = torch.load('mnist_model.pth')

@app.route('/predict', methods=['POST'])
def predict():
    data = request.json
    tensor_input = torch.tensor(data['input']).view(-1, 28*28)
    prediction = model(tensor_input).argmax().item()
    return jsonify({'prediction': int(prediction)})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Example input / output:**

```text
Input:
{
  "data": [
    [0.1, 0.2, ... , 0.9],
    ...
    [0.8, 0.7, ... , 0.3]
  ]
}

Output:
{
  "predictions": [0, 6, ..., 4]
}
```

**Data:**

```json
{
  "dataset": {
    "type": "MNIST",
    "shape": [64, 1, 28, 28],
    "size": 60000,
    "features": ["pixel_values"],
    "labels": ["class"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Training
8. Use GPUs?
9. High-Performance Compute
10. Limited Compute
11. Save Model Weights
12. Save Model Weights
13. Serving
14. Deploy on Cloud?
15. Dedicated Inference Instances
16. Use Local Resources
17. Handle High Volume Requests
18. Limited Scalability
19. High Availability
20. Reliability
21. Cost Optimization
22. Scalability
23. Deploy Model?
24. Distribute to Clients
25. Save for Later Use
26. Ensure Real-Time Predictions
27. Store and Update Manually

