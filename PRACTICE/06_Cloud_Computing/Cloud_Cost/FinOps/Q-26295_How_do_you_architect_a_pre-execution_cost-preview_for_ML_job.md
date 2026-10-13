---
type: "practice"
difficulty: "Deep"
---

# How do you architect a pre-execution cost-preview for ML jobs so an engineer sees the estimated GPU-hours and dollar cost of a training run or a large batch-inference job before submitting? What can you actually estimate ahead of time, and how do you handle the uncertainty?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost as a Non-functional Requirement](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_as_a_Non-functional_Requirement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a pre-execution cost-preview system for ML jobs, we need to focus on estimating GPU-hours and associated costs before an engineer submits their job. This involves understanding what can be precisely estimated, handling uncertainties, and ensuring the system is both efficient and user-friendly.

The core idea is to break down the problem into manageable parts: gathering input parameters from the user, calculating resource requirements, and then converting these into cost estimates based on current cloud pricing models. The key challenge lies in accurately predicting the job's runtime while accounting for variability factors such as data distribution and model complexity.

Let’s delve deeper:

Gathering Input Parameters: We need to collect essential information from the user, including:

Model size
Batch size
Data dimensions (e.g., number of rows, features)
Training epochs

Calculating Resource Requirements:

Data Loading Time: Estimate based on data size and network speed.
Inference/Training Time per Batch: This depends heavily on the model architecture and hardware specifications. For instance, a smaller model might train faster than a larger one with more parameters.
Total GPU-Hours: Sum up batch processing times over all epochs.

Cost Estimation:

Use historical data or averages to estimate cost per hour for different GPU types (e.g., Tesla V100, A100).
Factor in any potential usage spikes and idle periods.

Handling Uncertainty:

Provide worst-case and best-case scenarios.
Allow the system to learn from actual job runs to refine estimates over time.
Implement conservative estimates by factoring in inefficiencies like data shuffling and synchronization overhead.

User Interface: Present a clear, actionable preview that includes GPU-hours, estimated cost, and any potential bottlenecks.

Now let’s reason through the solution step-by-step from first principles:

Step 1: Input Collection

Engineers should input details such as:

Model configuration (number of layers, parameters)
Batch size
Dataset dimensions
Number of epochs
Step 2: Estimating Data Loading Time

The time to load data can be estimated using the formula: [ \text{Data Load Time} = \frac{\text{Total Data Size}}{\text{Network Speed (bps)}} \times \text{Factor (for compression, parallelism)} ]

Step 3: Calculating Inference/Training Time

For each batch, we can estimate time using: [ \text{Batch Processing Time} = \left( \frac{\text{Total Computation Time}}{\text{Batch Size}} \right) + \text{Overhead (for data transfer and synchronization)} ]

Step 4: Total GPU-Hours Calculation

Sum up the batch processing times over all epochs: [ \text{Total GPU-Hours} = \sum_{i=1}^{n} (\text{Batch Processing Time}_i) \times n ]

Step 5: Cost Estimation

Convert GPU-hours to cost using current pricing models. For example, if a Tesla V100 costs $1 per hour: [ \text{Estimated Cost} = \text{Total GPU-Hours} \times \text{Cost Per Hour} ]

Handling Uncertainty
Worst-case Scenario: Double the estimated time and cost.
Best-case Scenario: Halve the estimated time and cost, but ensure this is adjusted based on historical data.
User Interface Design

The system should present:

Estimated GPU-hours
Estimated cost in dollars
Potential bottlenecks (e.g., slow data loading, inefficient model)
Comparison with previous job runs

#### Closing Summary
+ Common Mistakes/Follow-ups

#### Common mistakes
include underestimating the complexity and variability of ML jobs. Engineers might overlook factors like data shuffling overhead or fail to account for network latency. To mitigate these issues:

Collect detailed historical data.
Implement conservative estimates.
Provide transparency into assumptions used in calculations.

#### Follow-up
questions could explore:

How do you handle job scheduling with multiple concurrent tasks?
What metrics would you use to evaluate the accuracy of your cost estimates?

**Comparison:**

```
Epoch	Batch Size	Computation Time (s)	Overhead (s)	Total Processing Time (s)
1	32	0.59	0.2	0.79
```

**Code:**

```python
# Example Python code for estimating GPU hours and costs

def estimate_data_load_time(total_data_size, network_speed_bps):
    return total_data_size / network_speed_bps * 1.5  # Factor for compression and parallelism

def batch_processing_time(model_size, batch_size):
    return (model_size / batch_size) + 0.2  # Overhead time in seconds

def calculate_total_gpu_hours(model_config, data_dimensions, num_epochs):
    total_gpu_hours = sum(batch_processing_time(model_config['size'], batch_size)
                          for epoch in range(num_epochs))
    return total_gpu_hours

def estimate_cost(total_gpu_hours, cost_per_hour=1.0):
    return total_gpu_hours * cost_per_hour
```

**Example input / output:**

```text
Input:
- Model: 24 layers, 50M parameters
- Data: 1GB, 100x100 dimensions
- Epochs: 100
- Batch Size: 32

Output:
- Estimated Data Load Time: 60 seconds
- Estimated Processing Time per Batch: 0.59 seconds
- Total GPU-Hours: 47.3 hours
- Estimated Cost: $47.3
```

**Data:**

```json
{
  "model": {
    "layers": 24,
    "parameters": 50M
  },
  "data": {
    "size": 1GB,
    "dimensions": [100, 100]
  },
  "epochs": 100,
  "batch_size": 32
}
```

**Flow:**

1. Model Configuration:::yes
2. Dataset Dimensions:::no
3. Input Parameters
4. Gather Data
5. Calculate Computation Time
6. Estimate Data Load Time
7. Sum Over Epochs
8. Calculate Total GPU-Hours
9. Cost Estimation
10. Report Results

