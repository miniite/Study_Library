---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python function that computes throughput (requests per second) from a list of per-second request counts and reports the peak second?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Percentage Change Calculations](<../../../../TOOLS/Python/Python_Language/Percentage_Change_Calculations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the throughput (requests per second) from a list of per-second request counts, we need to determine the maximum requests in any single second, as this will give us the peak throughput. Here’s how you would approach this problem step-by-step.

Firstly, the question is essentially asking for finding the maximum value in a list of integers, where each integer represents the number of requests in that particular second. The key idea is to iterate through the list and keep track of the highest count seen so far.

#### Assumptions

The input list will not be empty.
The input list contains non-negative integers (since negative request counts don't make sense in this context).

Let's break down the solution from first principles:

Initialize Variables:

We need a variable to keep track of the peak throughput (peak_throughput), which we initialize to 0.
A variable to store the current index, starting at 0.

Iterate Through the List:

For each element in the list (representing requests per second), compare it with peak_throughput.
If the current request count is higher than peak_throughput, update peak_throughput.

Return the Result:

After iterating through all elements, return the value of peak_throughput.

Here’s a step-by-step implementation in Python:

PYTHON
def compute_peak_throughput(request_counts):
    if not request_counts:  # Check if the list is empty
        raise ValueError("The input list cannot be empty")

    peak_throughput = 0

    for count in request_counts:
        if count > peak_throughput:
            peak_throughput = count

    return peak_throughput

This function iterates through each element of request_counts, updates the peak_throughput variable as needed, and finally returns the highest value encountered.

