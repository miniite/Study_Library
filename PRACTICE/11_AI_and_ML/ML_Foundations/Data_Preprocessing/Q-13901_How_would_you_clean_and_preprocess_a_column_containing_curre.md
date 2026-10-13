---
type: "practice"
difficulty: "Intermediate"
---

# How would you clean and preprocess a column containing currency values like '$1,200.50' stored as strings?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Cleaning Text and Numeric Fields](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cleaning_Text_and_Numeric_Fields.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To clean and preprocess a column containing currency values stored as strings such as "$1,200.50", we need to perform several steps:

Extract Numerical Value: Remove any non-numeric characters.
Handle Decimal Points: Convert the extracted string into a numerical format with the correct decimal point position.
Normalize Currency Symbols: Ensure that currency symbols are consistent if present.

The goal is to convert these strings into a clean, numeric format suitable for further analysis or modeling. Let's break this down step-by-step.

**Comparison:**

| index | original_value | cleaned_value |
|-------|----------------|---------------|
| 0     | $1,200.50      | 1200.5        |
| 1     | £987.65        | 987.65        |
| 2     | $4,321.89      | 4321.89       |
| 3     | €1,567.23      | 1567.23       |

**Code:**

```python
import pandas as pd

# Sample data
data = {'currency_values': ['$1,200.50', '£987.65', '$4,321.89', '€1,567.23']}
df = pd.DataFrame(data)
```

**Example input / output:**

```

Given the input DataFrame df, we will perform the following operations:

PYTHON
Copy
# Step 1: Remove currency symbols and commas
cleaned_values = df['currency_values'].str.replace('[\$,£€]', '', regex=True).str.replace(',', '')

# Step 2: Convert to float, handling potential errors gracefully
try:
    cleaned_values = cleaned_values.astype(float)
except ValueError as e:
    print(f"Conversion error: {e}")

print(cleaned_values)

Output:

CODE
Copy
0    1200.5
1     987.65
2    4321.89
3    1567.23
Name: currency_values, dtype: float64
```

**Data:**

```json
{
  "currency_values": ["$1,200.50", "£987.65", "$4,321.89", "€1,567.23"]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Start
6. Is currency symbol present?
7. Remove currency symbol andcommas
8. Convert to float
9. Save cleaned values
10. Error handling
11. Save original value as is

