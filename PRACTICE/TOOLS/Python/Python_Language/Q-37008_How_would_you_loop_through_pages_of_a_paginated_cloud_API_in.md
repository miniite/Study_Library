---
type: "practice"
difficulty: "Intermediate"
---

# How would you loop through pages of a paginated cloud API in Python to collect all results across pages?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Parsing JSON Safely](<../../../../TOOLS/Python/Python_Language/Parsing_JSON_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, let's first summarize the core concept and then break down the solution step by step.

TL;DR: We need to loop through multiple pages of a paginated API response in Python, collecting all results into a single list or data structure. This involves making repeated API calls until we have exhausted all pages, handling potential errors, and managing pagination tokens efficiently.

#### Triaging the question

The interviewer is probing your understanding of working with APIs in Python, especially those that are paginated.
We need to consider error handling, pagination mechanisms (like nextToken or pageToken), and managing API rate limits.

#### Assumptions
The API returns a JSON response per page, and we have the necessary credentials and access.

#### First-principles solution

Understand the API Response Structure: Typically, paginated APIs return a JSON structure with fields like items, nextToken, or similar pagination-related tokens.
Set Up Basic Variables:
PYTHON
import requests

base_url = 'https://example-api.com/data'
headers = {'Authorization': 'Bearer YOUR_API_KEY'}
all_results = []
next_token = None
Loop Through API Pages: Use a while loop to iterate through the pages.
Make API Call:
PYTHON
def fetch_data(url, headers, params=None):
    response = requests.get(url, headers=headers, params=params)
    if response.status_code != 200:
        raise Exception(f"API call failed with status code {response.status_code}")
    return response.json()
Process Each Page:
PYTHON
while next_token is not None:
    url = f"{base_url}?nextToken={next_token}"
    page_data = fetch_data(url, headers)
    all_results.extend(page_data['items'])
    next_token = page_data.get('nextToken', None)  # Check for the presence of a new token
Handle Edge Cases:
What if there is no nextToken returned?
How do we handle rate limiting or other API-specific constraints?

#### Closing summary
and common mistakes:

Ensure you correctly identify and use pagination tokens from the API response.
Handle potential errors in a robust manner, such as checking status codes and retrying failed requests.
Manage memory efficiently if dealing with large datasets.

Common follow-ups might include discussing how to handle rate limits, concurrency, or using asynchronous methods for better performance.

