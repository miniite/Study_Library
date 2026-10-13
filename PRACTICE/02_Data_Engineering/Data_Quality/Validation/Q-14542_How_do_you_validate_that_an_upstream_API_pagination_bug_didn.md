---
type: "practice"
difficulty: "Deep"
---

# How do you validate that an upstream API pagination bug didn't silently drop the last page of records during ingestion, given each page individually looks valid?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Validating Loads and Deduplication](<../../../../02_Data_Engineering/Data_Quality/Validation/Validating_Loads_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that an upstream API pagination bug hasn’t silently dropped the last page of records, we need to implement a robust validation strategy that goes beyond simple data checks on individual pages. This involves understanding the nature of the pagination, the schema of the data, and implementing comprehensive quality checks.

#### Triaging the Question

The core issue here is ensuring the completeness of the dataset when dealing with paginated APIs, which typically return multiple pages of records sequentially. The key points to consider are:

Pagination Logic: How does the API handle pagination? Does it support range queries or only fetches from the end?
Data Schema: What structure do the records have, and what fields can be used for validation?

#### Cons
istency Checks: Are there any consistent patterns or summaries that should hold true across all pages?

#### Assumptions
The API provides a consistent way to request the last page of records.
There are no other bugs affecting data integrity besides pagination.
Step-by-Step Solution

Understand Pagination Logic

Determine if the API allows requesting the last page directly or if you need to fetch pages sequentially until the last one is reached.

Fetch All Pages Sequentially

If the API supports range queries, request a series of pages starting from the first and moving to the last.
If it only supports fetching records in sequence, manually track the page numbers.

Store Page Responses

Collect all responses in an intermediate storage system (e.g., a database table or a data lake).

Calculate Total Expected Records

Use metadata provided by the API to calculate the total number of expected records.
If no metadata is available, use heuristics based on historical data.

Validate Individual Pages

Perform basic validation checks (e.g., ensure all fields are non-null, check for valid formats) on each page.

Check for Missing Records

Compare the total count of records collected from all pages with the expected number.
Identify discrepancies and flag any missing records.

Summarize Results

Generate a report summarizing which pages were successfully fetched, how many records were collected, and whether there are any missing records.

#### Example
Workflow
PYTHON
# Pseudocode for fetching data from an API with pagination
def fetch_data_from_api(page=1):
    response = requests.get(f"https://api.example.com/data?page={page}")
    return response.json()

def validate_pagination():
    total_records_expected = 1000  # Example expected number of records based on historical data
    collected_records = []

    current_page = 1
    while True:
        page_data = fetch_data_from_api(current_page)
        if not page_data['records']:
            break
        collected_records.extend(page_data['records'])
        current_page += 1

    actual_count = len(collected_records)
    if actual_count < total_records_expected:
        print(f"Warning: Missing records. Expected {total_records_expected}, got {actual_count}.")

    # Further validation steps can be added here

# Example usage
validate_pagination()

#### Closing Summary
and Common Mistakes/Follow-Ups

To ensure the data quality, it is crucial to validate not only individual pages but also the overall completeness of the dataset. Common mistakes include:

Overlooking metadata or historical data that could provide insights into expected record counts.
Failing to implement comprehensive validation checks beyond basic field integrity.

For follow-ups, consider discussing how to handle different API implementations, caching strategies, and integration with downstream systems that rely on complete data sets.

**Comparison:**

```
Page	Records Fetched
1	2
2	2
last_page	0
```

**Code:**

```python
# Pseudocode for fetching data from an API with pagination
def fetch_data_from_api(page=1):
    response = requests.get(f"https://api.example.com/data?page={page}")
    return response.json()

def validate_pagination():
    total_records_expected = 1000  # Example expected number of records based on historical data
    collected_records = []

    current_page = 1
    while True:
        page_data = fetch_data_from_api(current_page)
        if not page_data['records']:
            break
        collected_records.extend(page_data['records'])
        current_page += 1

    actual_count = len(collected_records)
    if actual_count < total_records_expected:
        print(f"Warning: Missing records. Expected {total_records_expected}, got {actual_count}.")

    # Further validation steps can be added here

# Example usage
validate_pagination()
```

**Example input / output:**

```

Input:

PYTHON
Copy
total_records_expected = 1000

Output:

PLAINTEXT
Copy
Warning: Missing records. Expected 1000, got 4.
```

**Data:**

```json
{
  "page": {
    "1": ["record1", "record2"],
    "2": ["record3", "record4"],
    "last_page": []
  }
}
```

**Flow:**

1. Yes
2. No
3. Discrepancy Detected
4. No Discrepancies
5. Start
6. Is API Range QuerySupported?
7. Fetch Data Sequentially
8. Calculate Last Page Number
9. Store Responses in StorageSystem
10. Store Responses in StorageSystem
11. E,F
12. Calculate Total RecordsCollected
13. Compare with Expected Count
14. Log Warning, Identify MissingRecords
15. Generate Summary Report

