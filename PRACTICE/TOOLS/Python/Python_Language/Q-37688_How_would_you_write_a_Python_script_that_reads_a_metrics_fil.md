---
type: "practice"
difficulty: "Basic"
---

# How would you write a Python script that reads a metrics file and produces a summary dict with min, max, mean, and count for a quick report?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Data Basics](<../../../../TOOLS/Python/Python_Language/Time-series_Data_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to create a Python script that can read a metrics file (assuming it's in a text format like CSV) and generate a summary dictionary containing the minimum value (min), maximum value (max), mean value (mean), and count of the values (count). This will help us quickly summarize a set of numerical data.

#### Triaging the Question

The interviewer is probing your understanding of basic file handling in Python, as well as your ability to perform arithmetic operations and use dictionary structures. The key assumptions are that the input file has numerical values (which we'll assume are space-separated), and it's formatted in a simple way for this example.

#### First-Principles Solution
Open and Read the File: We need to open and read the file line by line.
Parse the Data: For each line, split the values and convert them to floats (assuming they are numbers).
Initialize Variables: Set up variables to keep track of min, max, a running total for the mean calculation, and a counter for counts.
Calculate Metrics:
Update min if we encounter a smaller value.
Update max if we encounter a larger value.
Add the current number to our sum.
Increment the count.
Compute Mean: After processing all values, compute the mean by dividing the total sum by the count.
Full Solution

Here’s how you can implement this in Python:

PYTHON
def summarize_metrics(filename):
    # Initialize variables
    min_val = float('inf')
    max_val = -float('inf')
    total_sum = 0
    value_count = 0

    with open(filename, 'r') as file:
        for line in file:
            values = [float(val) for val in line.split()]
            for val in values:
                # Update min and max
                if val < min_val:
                    min_val = val
                if val > max_val:
                    max_val = val

                # Update sum and count
                total_sum += val
                value_count += 1

    # Calculate mean
    mean_val = total_sum / value_count if value_count != 0 else None

    # Return the summary dictionary
    return {
        'min': min_val,
        'max': max_val,
        'mean': mean_val,
        'count': value_count
    }

# Example usage
summary = summarize_metrics('metrics.txt')
print(summary)

#### Closing Summary
and Common Mistakes/Follow-Ups

This script reads a file, processes the values, and returns a dictionary with the required metrics. A common mistake could be forgetting to handle empty lines or non-numeric data. Another potential issue is not initializing min and max properly, which can lead to incorrect results if the input file starts with very large or small numbers.

If asked for improvements, you might consider:

Adding error handling for invalid inputs (non-numeric data).
Handling larger files by processing line-by-line instead of storing all values in memory.
Using a more sophisticated approach like NumPy for statistical calculations.

