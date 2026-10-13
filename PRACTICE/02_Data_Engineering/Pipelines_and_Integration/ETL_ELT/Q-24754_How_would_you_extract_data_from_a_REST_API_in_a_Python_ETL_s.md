---
type: "practice"
difficulty: "Intermediate"
---

# How would you extract data from a REST API in a Python ETL script and handle pagination?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ ETL vs ELT Design](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/ETL_vs_ELT_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract data from a REST API in a Python ETL script while handling pagination, we need to ensure that our script can make multiple API calls, retrieve all pages of results, and process each piece of data correctly. We'll start by discussing the core concept and then delve into the steps and code details.

#### Triage

The question is asking for a method to extract data from a REST API in an ETL (Extract, Transform, Load) script while considering pagination. The interviewer is probing your understanding of making HTTP requests, handling responses, managing states between multiple calls, and ensuring robust error handling.

Core Concept and Assumptions
API Details: We need the base URL and endpoint for the API.
Pagination Parameters: Most APIs use parameters like page, limit, or similar to manage pagination (e.g., /api/data?page=1&limit=10).
Data Processing: Once we retrieve data, it will be transformed and loaded into a database or another storage system.

#### First-Principles Solution

The first step is to define the function that makes HTTP requests and handles pagination. We'll use Python's requests library for making API calls.

Step 1: Define the Base Function

We need a reusable function to fetch data from an API with pagination support.

PYTHON
import requests

def fetch_api_data(base_url, params=None):
    headers = {
        'Content-Type': 'application/json'
    }

    while True:
        response = requests.get(base_url, headers=headers, params=params)

        if not response.ok:
            print(f"Error: {response.status_code}")
            return

        data = response.json()
        yield from data  # Yield each item in the page

        next_page = get_next_page_link(data)  # Function to extract next page URL
        if not next_page:
            break

        params['page'] = next_page.split('=')[1]
Step 2: Extract Next Page Link

We need a helper function to parse and retrieve the next link from the response.

PYTHON
def get_next_page_link(data):
    for link in data.get('links', []):
        if 'rel' in link and link['rel'] == 'next':
            return link['href']

    return None
Step 3: Process Data

After fetching all data, we process it according to the ETL requirements.

PYTHON
def transform_and_load(data):
    transformed_data = []
    for item in data:
        # Transform logic here (e.g., clean, enrich)
        transformed_item = {
            'id': item['id'],
            'name': item['name'].upper(),
            # Add more transformations as needed
        }
        transformed_data.append(transformed_item)

    # Load into database or another storage system
    save_to_database(transformed_data)

def save_to_database(data):
    # Code to insert data into the target database or storage
    pass

# Main script entry point
if __name__ == '__main__':
    base_url = 'https://api.example.com/data'
    params = {
        'limit': 10,
        'page': 1
    }

    for item in fetch_api_data(base_url, params):
        transform_and_load(item)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we designed a Python script that makes API calls to handle pagination using the requests library. The script ensures robust error handling and data transformation before loading it into a storage system.

#### Common Mistakes

Lack of Error Handling: Failing to check for HTTP errors.
Overwriting Parameters: Not updating the page parameter correctly in subsequent calls.
Incorrect Data Transformation: Neglecting to properly transform data according to business requirements.
Database Integration: Failing to integrate with a database or storage system.

If asked follow-ups, we could discuss alternatives like using libraries such as httpx for better performance and reliability, or scaling the solution by adding concurrency if dealing with large datasets.

**Code:**

```python
import requests

def fetch_api_data(base_url, params=None):
    headers = {
        'Content-Type': 'application/json'
    }

    while True:
        response = requests.get(base_url, headers=headers, params=params)

        if not response.ok:
            print(f"Error: {response.status_code}")
            return

        data = response.json()
        yield from data  # Yield each item in the page

        next_page = get_next_page_link(data)  # Function to extract next page URL
        if not next_page:
            break

        params['page'] = next_page.split('=')[1]

def transform_and_load(data):
    transformed_data = []
    for item in data:
        # Transform logic here (e.g., clean, enrich)
        transformed_item = {
            'id': item['id'],
            'name': item['name'].upper(),
            # Add more transformations as needed
        }
        transformed_data.append(transformed_item)

    # Load into database or another storage system
    save_to_database(transformed_data)

def get_next_page_link(data):
    for link in data.get('links', []):
        if 'rel' in link and link['rel'] == 'next':
            return link['href']

    return None

def save_to_database(data):
    # Code to insert data into the target database or storage
    pass

# Main script entry point
if __name__ == '__main__':
    base_url = 'https://api.example.com/data'
    params = {
        'limit': 10,
        'page': 1
    }

    for item in fetch_api_data(base_url, params):
        transform_and_load(item)
```

**Flow:**

1. Yes
2. No
3. Fetch API Data
4. Is Next Page?
5. Fetch Next Page
6. Transform and Load Data

