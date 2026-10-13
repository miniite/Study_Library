---
type: "practice"
difficulty: "Intermediate"
---

# How would you right-size a GPU instance for serving a mid-size model, given you have not yet profiled its real memory and throughput needs?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ GPU Sharing](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To right-size a GPU instance for serving a mid-sized ML model without having fully profiled its specific memory and throughput requirements, we need to follow a structured approach that balances cost efficiency with performance. We'll start by understanding the key parameters required (such as the model size, expected traffic, and desired latency) and then leverage cloud provider recommendations and benchmarks to make an informed decision.

First, let's break down what "right-sizing" means in this context:

Memory: The total VRAM available on the GPU must be sufficient for the model.
Throughput: The GPU should handle the expected number of requests per second without bottlenecking.
Latency: The time taken to process and return a prediction is critical, especially under load.

Now, let's walk through the steps to determine the appropriate GPU instance:

Understand Model Requirements:

Model Size: Typically, larger models require more VRAM. For example, an Inception model might fit in 8GB of VRAM, while a BERT model could require over 16GB.
Expected Traffic: Estimate the number of requests per minute or hour that your service will handle.

Benchmarking and Cloud Provider Recommendations:

AWS EC2: Use services like Amazon SageMaker for benchmarking different GPU instances.
Google Cloud: Use Vertex AI to assess performance on various GPUs.
Azure ML: Use Azure’s recommendation tools or test different VM sizes.

Initial Instance Selection:

Start with a medium-sized instance, such as an AWS P3.8xlarge (with 24GB VRAM) or equivalent from other providers.
This gives you some headroom for larger models and higher traffic if needed.

Performance Testing:

Deploy the model on the selected GPU instance.
Run a series of test scenarios, including peak load testing to see how well the system handles high traffic.
Monitor performance metrics such as inference time, memory usage, and overall throughput.

Cost Considerations:

Evaluate the cost of different instances over a period (e.g., 24 hours).
Compare this cost with the cost benefits of using more powerful or smaller instances.

Adjustment Based on Monitoring:

If the initial instance is underutilized, consider downsizing.
If it consistently hits bottlenecks, upsize to a larger instance type.

Final Decision:

Choose an instance that provides adequate performance for both current and future growth needs without over-provisioning.

**Comparison:**

```
Instance Type	VRAM (GB)	Max Instances	Cost per Hour
p3.2xlarge	12	4	$0.89
p3.8xlarge	24	2	$2.65
g4dn.xlarge	16	10	$0.70
```

**Code:**

```python
# Example Python script to run initial benchmark tests on AWS SageMaker

import boto3

def test_sagemaker_instance(instance_type):
    client = boto3.client('sagemaker')

    # Deploy model
    response = client.create_endpoint_config(
        EndpointConfigName='my-endpoint',
        ProductionVariants=[{
            'InstanceType': instance_type,
            'InitialVariantWeight': 1.0,
            'ModelName': 'my-model'
        }]
    )

    endpoint_name = response['EndpointConfigArn'].split('/')[-1]

    # Test inference
    client.create_endpoint(
        EndpointName=endpoint_name,
        EndpointConfigurationName='my-endpoint'
    )

    # Monitor and log performance metrics
    print(f"Testing {instance_type} instance...")

test_sagemaker_instance('p3.8xlarge')
```

**Example input / output:**

```text
Input:
- Model chosen: BERT Base

Output:
- P3.8xlarge instance selected for testing
- Initial performance metrics collected (inference time, memory usage)
- Test results indicate high throughput with acceptable latency
```

**Data:**

```json
{
  "model_size": {
    "inception_v3": 2.0,
    "bert_base": 15.0
  },
  "expected_requests_per_minute": 600,
  "current_load_factor": 0.7
}
```

**Flow:**

1. Identify Model Requirements
2. Choose Initial Instance
3. Deploy and Monitor
4. Test Performance
5. Adjust if Necessary
6. Final Decision on Right-Sizing

