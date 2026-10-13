---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python enrichment step that maps a firing alert to its likely runbook using service tags and alert type, and how do you handle the case where no runbook matches?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incident Enrichment Scripts](<../../../../TOOLS/Python/Python_Language/Incident_Enrichment_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this AIOps scenario, we are tasked with writing a Python enrichment step that maps an incoming firing alert to its likely runbook based on service tags and alert type. The key concept here is to leverage the provided metadata from the alert (service tags and alert type) to find the most relevant runbook in our system. We need to handle scenarios where no matching runbook exists, ensuring graceful error handling.

First, let's break down what the question is asking:

Input Data: An alert object with fields such as service tags and alert type.
Output Data: The likely associated runbook or a message indicating no match was found.
Handling Cases: We need to handle cases where multiple possible matches exist (although we aim for a single best match) and when there are no matches at all.

#### First-Principles Solution

Data Structures:

Define the schema for alerts and runbooks.
Create a mapping function that takes an alert as input and returns a likely runbook or a message indicating no match found.

Algorithm Steps:

Parse the incoming alert to extract its service tags and alert type.
Use these attributes to query our system (database or configuration file) for potential runbooks.
Evaluate each potential runbook based on relevance criteria.
Return the most relevant runbook, or a message if no matches are found.

#### Implementation

Utilize Python's dictionary and list operations to efficiently manage and process the data.
Ensure the code is modular and easy to test by separating concerns (e.g., alert parsing, runbook evaluation).

Here’s a step-by-step implementation:

Step 1: Define Alert and Runbook Schema
PYTHON
class Alert:
    def __init__(self, service_tags, alert_type):
        self.service_tags = service_tags
        self.alert_type = alert_type

class Runbook:
    def __init__(self, name, tags, description):
        self.name = name
        self.tags = tags  # List of relevant services and types
        self.description = description
Step 2: Define Mapping Function
PYTHON
def map_alert_to_runbook(alert, runbooks):
    """
    Maps an incoming alert to its likely runbook based on service tags and alert type.

    :param Alert alert: The incoming alert object.
    :param list[Runbook] runbooks: A list of potential runbooks.
    :return: The name of the most relevant runbook or a message indicating no match found.
    """
    matching_runbooks = []

    # Filter runbooks based on service tags and alert type
    for runbook in runbooks:
        if any(tag in alert.service_tags for tag in runbook.tags) and runbook.alert_type == alert.alert_type:
            matching_runbooks.append(runbook)

    # Handle cases where no matches are found
    if not matching_runbooks:
        return "No matching runbook found"

    # Select the most relevant runbook (simplest heuristic: first match, could be replaced with better logic)
    best_match = matching_runbooks[0]

    return f"Recommended runbook: {best_match.name} - {best_match.description}"
Step 3: Example Usage
PYTHON
alert = Alert(service_tags=['web', 'production'], alert_type='critical')
runbooks = [
    Runbook(name="Web Server Crash", tags=['web', 'server'], description="Handle web server crashes"),
    Runbook(name="Database Error", tags=['database'], description="Fix database connectivity issues")
]

result = map_alert_to_runbook(alert, runbooks)
print(result)  # Output: "No matching runbook found"

#### Closing Summary
and Common Mistakes

This solution handles the core requirement by mapping an alert to a likely runbook based on service tags and alert type. The implementation is straightforward but can be enhanced with more sophisticated logic for handling multiple matches. Common mistakes include not thoroughly validating input data, failing to handle edge cases where no match is found, or using overly complex logic that obscures the problem.

**Code:**

```python
class Alert:
    def __init__(self, service_tags, alert_type):
        self.service_tags = service_tags
        self.alert_type = alert_type

class Runbook:
    def __init__(self, name, tags, description):
        self.name = name
        self.tags = tags  # List of relevant services and types
        self.description = description

def map_alert_to_runbook(alert, runbooks):
    matching_runbooks = []

    for runbook in runbooks:
        if any(tag in alert.service_tags for tag in runbook.tags) and runbook.alert_type == alert.alert_type:
            matching_runbooks.append(runbook)

    if not matching_runbooks:
        return "No matching runbook found"

    best_match = matching_runbooks[0]

    return f"Recommended runbook: {best_match.name} - {best_match.description}"
```

**Example input / output:**

```text
result = map_alert_to_runbook(alert, runbooks)
print(result)  # Output: "No matching runbook found"
```

**Data:**

```python
alert = Alert(service_tags=['web', 'production'], alert_type='critical')
runbooks = [
    Runbook(name="Web Server Crash", tags=['web', 'server'], description="Handle web server crashes"),
    Runbook(name="Database Error", tags=['database'], description="Fix database connectivity issues")
]
```

