---
type: "practice"
difficulty: "Intermediate"
---

# What is data partitioning by date, and how would a Python job write output files into folders like year=2026/month=07 for efficient querying?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Comparing Time Windows](<../../../../TOOLS/Python/Python_Language/Comparing_Time_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Data partitioning by date involves organizing data based on specific date-related criteria to optimize storage, retrieval, and querying of the data. In this case, we want a Python job that writes output files into folders structured as year=2026/month=07 for efficient querying. This approach allows us to quickly locate and access data from a particular time period without scanning through an entire dataset.

To achieve this in Python, we'll need to:

Determine the current date or desired date.
Extract the year and month components from the date.
Create the appropriate folder structure if it doesn't already exist.
Write the output files into these folders.

Let's go through each step using first principles:

#### Triage

The question is asking us to implement a Python function that handles data partitioning by date, specifically creating and writing files into hierarchical folders based on year and month. The key points are:

We need to extract date components.
Ensure the folder structure exists before writing.
Write output files with proper naming.

#### First Principles Solution
Date Extraction: Use Python's datetime module to get the current date or a specified date.
Folder Creation: Check if the required directory structure exists; create it if necessary.
File Writing: Write data into files placed in the correct folder.

Here’s how we can implement this:

PYTHON
import os
from datetime import datetime

def write_partitioned_files(data, date_string=None):
    # If no specific date is given, use current date
    if not date_string:
        date_string = datetime.now().strftime('%Y-%m')

    year, month = map(int, date_string.split('-'))

    # Define the base directory for partitioning data
    base_dir = 'data'

    # Construct full path to the target folder
    folder_path = os.path.join(base_dir, str(year), f"month={month}")

    # Ensure the directory exists
    if not os.path.exists(folder_path):
        os.makedirs(folder_path)

    # Example: Write data into a file named after the month and year
    filename = f"data_{year}_{month}.txt"
    filepath = os.path.join(folder_path, filename)

    with open(filepath, 'w') as file:
        for item in data:
            file.write(f"{item}\n")

    print(f"Data written to: {filepath}")

# Example usage
data_to_write = ["2026-07-01", "2026-07-02"]
write_partitioned_files(data_to_write)

#### Closing Summary
and Common Mistakes/Follow-Ups

This script demonstrates how to write data into partitioned directories based on date. Key points include:

Using datetime for date handling.
Ensuring directory existence before writing files.
Proper file naming conventions.

#### Common Mistakes

Not checking if the folder exists, leading to potential errors when trying to write to non-existent paths.
Incorrect file path construction, which can result in file not being found or written incorrectly.
Using incorrect date format, causing issues during date extraction and comparison.

For follow-ups, you might be asked about:

#### Performance considerations
for large datasets.
Handling multiple types of data (e.g., CSV vs JSON).
Security implications when writing to files.

