---
type: "practice"
difficulty: "Intermediate"
---

# What is a z-score, and how would you write Python code that flags a metric reading as anomalous if its z-score exceeds 3?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Simple Anomaly Detection](<../../../../TOOLS/Python/Python_Language/Simple_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with anomalies in data, particularly time series metrics, the z-score is a common statistical measure used to identify outliers. A z-score represents how many standard deviations a value is from its mean. In this context, we flag a metric reading as anomalous if its z-score exceeds 3, which corresponds to approximately a 0.13% chance of occurring by random chance under the assumption that data follows a normal distribution.

To implement this in Python, we'll start by calculating the z-score for each data point and then flagging any points where the absolute value of the z-score is greater than 3.

#### Triage

The question asks us to write Python code that flags metric readings as anomalous based on their z-scores. We need to understand:

The formula for calculating a z-score.
How to apply this formula to each data point in our dataset.
Setting a threshold (z > 3) to identify anomalies.

#### First-Principles Solution

The z-score is calculated using the following formula:

[ z = \frac{x - \mu}{\sigma} ]

where:

( x ) is the metric value,
( \mu ) is the mean of the dataset,
( \sigma ) is the standard deviation of the dataset.

To flag a reading as anomalous, we check if the absolute z-score exceeds 3. Here’s how to implement this in Python:

Calculate Mean and Standard Deviation: These are essential statistics needed for computing z-scores.
Compute Z-Score for Each Data Point: Using the mean and standard deviation calculated from step 1.
Flag Anomalies: Identify data points where the absolute value of the z-score is greater than 3.
Code Implementation

Let's write the Python code to achieve this:

PYTHON
import numpy as np

def calculate_z_scores(data):
    # Step 1: Calculate mean and standard deviation
    mu = np.mean(data)
    sigma = np.std(data, ddof=1)  # Using sample standard deviation

    # Step 2: Compute z-scores for each data point
    z_scores = [(x - mu) / sigma for x in data]

    # Step 3: Flag anomalies (absolute value of z-score > 3)
    anomalies = [data[i] if abs(z_scores[i]) > 3 else None for i in range(len(data))]

    return z_scores, anomalies

# Example usage
metric_data = [10.2, 15.6, 20.4, 25.8, 30.1, 35.2, 40.7, 45.9, 50.3, 55.2]
z_scores, anomalies = calculate_z_scores(metric_data)

print("Z-scores:", z_scores)
print("Anomalies flagged (if any):", anomalies)

#### Closing Summary
and Common Mistakes

The code above correctly implements the z-score calculation and anomaly detection based on a threshold of 3. However, here are some common pitfalls to avoid:

Ensure that your data meets the assumption of normality before applying z-scores.
Be cautious with small datasets, as standard deviation calculations may not be robust.
The choice of 3 as the threshold is arbitrary; depending on the application, a different threshold might be more suitable.

#### Common Mistakes
Incorrect Standard Deviation Calculation: Using population standard deviation (ddof=0) instead of sample standard deviation (ddof=1).
Not Handling Data Size Appropriately: Small datasets can lead to unreliable z-score calculations.
Improper Threshold Setting: The threshold of 3 is a common choice but might not be ideal for all applications.

