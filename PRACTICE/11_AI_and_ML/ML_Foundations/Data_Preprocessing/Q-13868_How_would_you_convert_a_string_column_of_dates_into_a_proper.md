---
type: "practice"
difficulty: "Basic"
---

# How would you convert a string column of dates into a proper datetime format in pandas?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Cleaning Text and Numeric Fields](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cleaning_Text_and_Numeric_Fields.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To convert a string column of dates into a proper datetime format using pandas, you need to ensure that the date strings are in a recognizable format and then use the pd.to_datetime() function to parse them correctly. This is crucial for further analysis or processing that requires accurate temporal data.

The key steps involve:

Identifying the correct date format.
Using pd.to_datetime() with appropriate arguments.
Handling any parsing errors gracefully.

Let's break it down from first principles:

#### Triaging the Question

The question asks how to convert a string column containing dates into a proper datetime format in pandas. The interviewer is probing your understanding of data manipulation and error handling within pandas. You need to make assumptions about the date formats you might encounter, such as YYYY-MM-DD, MM/DD/YYYY, or others.

#### First-Principles Solution

Identify the Date Format: Determine the exact format of the dates in your string column. Common formats include:

%Y-%m-%d (e.g., "2023-10-05")
%m/%d/%Y (e.g., "10/05/2023")
%d %b %Y (e.g., "5 Oct 2023")

Use pd.to_datetime(): Apply the pd.to_datetime() function with the correct format string to parse the dates correctly.

Handle Parsing Errors: Use the errors='coerce' argument to replace invalid parsing with NaT, which stands for "Not a Time."

Check and Clean Data: Verify that the conversion was successful by checking if any values are still in string format.

Code Example

Here’s how you would implement this:

PYTHON
import pandas as pd

# Sample DataFrame
data = {'dates': ['2023-10-05', '10/05/2023', '5 Oct 2023', 'Invalid Date']}
df = pd.DataFrame(data)

# Convert to datetime with the correct format for each type of date string
df['dates'] = pd.to_datetime(df['dates'], errors='coerce')

print(df)
Output Example
PLAINTEXT
         dates
0 2023-10-05
1 2023-10-05
2 2023-10-05
3       NaT

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key takeaway is to use pd.to_datetime() with the appropriate format string and handle parsing errors. Common mistakes include:

Not specifying the correct date format, leading to incorrect conversions.
Ignoring parsing errors, which can introduce missing values (NaT) that may need further handling.

For follow-ups, you might be asked about dealing with ambiguous formats or large datasets where performance is critical.

**Code:**

```python
import pandas as pd

# Sample DataFrame
data = {'dates': ['2023-10-05', '10/05/2023', '5 Oct 2023', 'Invalid Date']}
df = pd.DataFrame(data)

# Convert to datetime with the correct format for each type of date string
df['dates'] = pd.to_datetime(df['dates'], errors='coerce')

print(df)
```

**Example input / output:**

```text
dates
0 2023-10-05
1 2023-10-05
2 2023-10-05
3       NaT
```

**Data:**

```python
data = {'dates': ['2023-10-05', '10/05/2023', '5 Oct 2023', 'Invalid Date']}
df = pd.DataFrame(data)
```

