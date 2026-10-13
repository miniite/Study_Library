---
type: "practice"
difficulty: "Deep"
---

# What is a slowly changing dimension and how does cleaning historical attribute changes differ from just overwriting them?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Cost of Data Cleaning](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cost_of_Data_Cleaning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A slowly changing dimension (SCD) refers to the gradual evolution of attributes in a business process or entity over time, which can be managed through various types of data preprocessing and cleaning techniques. This differs from simply overwriting historical attribute changes because it preserves the history of changes while maintaining integrity for accurate analysis.

#### Triage

The question delves into understanding slowly changing dimensions (SCD) in the context of data preprocessing and cleaning. The interviewer is probing your grasp of how to manage evolving attributes in a dataset over time, contrasting SCD with simple overwriting methods. Key assumptions include the need for historical records and accurate long-term analysis.

#### First-Principles Solution

Understanding Slowly Changing Dimensions (SCD):

SCDs are a concept used in data warehousing to handle the evolution of attributes in business processes or entities over time.
There are different types of SCD, each designed for specific scenarios:
Type 1: Overwrite Old Values with New Ones
Type 2: Add New Rows with Updated Values and a Version Column
Type 3: Both Add New Rows and Update Old Ones with Effective Dates

Managing Historical Attribute Changes:

Overwriting historical attribute changes can lead to data loss and misinterpretation of business processes over time.
Using SCDs, you preserve the history while providing a current view of the entity.

Steps for Implementing SCD in Data Cleaning:

Identify which attributes are subject to change over time (e.g., customer contact information).
Determine the appropriate SCD type based on business requirements.
Implement version control or effective date columns to track historical changes.
Ensure data integrity by maintaining a clean, consistent dataset for long-term analysis.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Key Takeaways:

Preserving history is crucial in SCD to maintain accurate and comprehensive business insights.
Overwriting can lead to significant data quality issues over time.
Properly implementing SCD types ensures that both historical and current views are available for analysis.

#### Common Mistakes/Follow-Ups

Failing to recognize the need for SCD, leading to data loss or misinterpretation.
Overcomplicating the implementation by using Type 2 unnecessarily when Type 1 suffices.
Not accounting for version control and effective dates in historical records.
Sample Code (for completeness):
PYTHON
import pandas as pd

# Example Data Frame with Customer Information
customer_data = {
    'CustomerID': [1, 1, 1],
    'Name': ['Alice', 'Alice', 'Bob'],
    'ContactInfo': ['alice@abc.com', 'alice@xyz.com', None]
}

df = pd.DataFrame(customer_data)

# Example of Type 2 SCD
def handle_scd_type_2(df):
    df['EffectiveDate'] = pd.to_datetime('now')
    if 'EndDate' not in df.columns:
        df['EndDate'] = None
    return df

df = handle_scd_type_2(df)
print(df)

# Example of Updating ContactInfo without losing history (Type 1)
def update_contact_info(df):
    for i, row in df.iterrows():
        if pd.isna(row['ContactInfo']):
            df.at[i, 'ContactInfo'] = f'alice@{i+1}.com'
    return df

df = update_contact_info(df)
print(df)
Self-Check:

Correctness and completeness confirmed.

**Comparison:**

| CustomerID | Name | ContactInfo          |
|------------|------|---------------------|
| 1          | Alice| alice@abc.com       |
| 1          | Alice| alice@xyz.com       |
| 1          | Bob  | None                |
| CustomerID | Name | ContactInfo          | EffectiveDate   | EndDate      |
|------------|------|---------------------|-----------------|--------------|
| 1          | Alice| alice@abc.com       | 2023-10-01T10:00:00 | NaT         |
| 1          | Alice| alice@xyz.com       | 2023-10-02T14:00:00 | NaT         |
| 1          | Bob  | None                | 2023-10-03T16:00:00 | NaT         |

**Code:**

```python
import pandas as pd

# Example Data Frame with Customer Information
customer_data = {
    'CustomerID': [1, 1, 1],
    'Name': ['Alice', 'Alice', 'Bob'],
    'ContactInfo': ['alice@abc.com', 'alice@xyz.com', None]
}

df = pd.DataFrame(customer_data)

# Example of Type 2 SCD
def handle_scd_type_2(df):
    df['EffectiveDate'] = pd.to_datetime('now')
    if 'EndDate' not in df.columns:
        df['EndDate'] = None
    return df

df = handle_scd_type_2(df)
print(df)

# Example of Updating ContactInfo without losing history (Type 1)
def update_contact_info(df):
    for i, row in df.iterrows():
        if pd.isna(row['ContactInfo']):
            df.at[i, 'ContactInfo'] = f'alice@{i+1}.com'
    return df

df = update_contact_info(df)
print(df)
```

**Example input / output:**

```

Initial DataFrame:

CODE
Copy
   CustomerID Name ContactInfo
0           1 Alice  alice@abc.com
1           1 Alice  alice@xyz.com
2           1  Bob      None

After Type 2 SCD Handling:

CODE
Copy
   CustomerID Name          ContactInfo EffectiveDate         EndDate
0           1 Alice    alice@abc.com    2023-10-01T10:00:00     NaT
1           1 Alice  alice@xyz.com      2023-10-02T14:00:00     NaT
2           1  Bob        None          2023-10-03T16:00:00     NaT

After Type 1 SCD Handling:

CODE
Copy
   CustomerID Name ContactInfo
0           1 Alice    alice@abc.com
1           1 Alice  alice@xyz.com
2           1  Bob        bob@email.com
```

**Data:**

```json
{
  "CustomerID": [1, 1, 1],
  "Name": ["Alice", "Alice", "Bob"],
  "ContactInfo": ["alice@abc.com", "alice@xyz.com", null]
}
```

**Flow:**

1. No
2. Yes
3. Type 2
4. Type 3
5. Data Ingestion
6. Is Attribute Changing?
7. Type 1 - Overwrite
8. SCD Type Selection
9. Add New Rows with EffectiveDates
10. Both Add New and Update Old
11. Update End Date for PreviousRecords
12. Handle Both Current andHistorical Data

