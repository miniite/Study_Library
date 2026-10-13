---
type: "practice"
difficulty: "Basic"
---

# What is an incident in operations, and how would you write a Python function that records an incident with a start time, severity, and description into a list?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Incident Enrichment Scripts](<../../../../TOOLS/Python/Python_Language/Incident_Enrichment_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps (Artificial Intelligence for IT Operations), an "incident" is typically defined as a disruption or unexpected event in the operations of a system that impacts service delivery. This could range from a user reporting a critical application issue to a network outage affecting multiple services.

Now, let's write a Python function to record such incidents into a list with details like start time, severity, and description.

First, we need to understand what each component of the incident is:

Start Time: The timestamp when the incident was first noticed.
Severity: How severe or critical the incident is (e.g., "critical," "major," "minor").
Description: A brief summary of what happened and any relevant details.

Here’s a step-by-step guide to writing this Python function:

Define a class Incident with attributes for start time, severity, and description.
Create a list to hold these incidents.
Write a function record_incident that takes the incident details as input and appends it to the list.

Let's go through each step in detail:

Step 1: Define an Incident Class
PYTHON
class Incident:
    def __init__(self, start_time, severity, description):
        self.start_time = start_time
        self.severity = severity
        self.description = description

Step 2: Create a List to Hold Incidents

We can use a list of Incident objects.

PYTHON
incidents_list = []
Step 3: Write the Function to Record an Incident

The function record_incident will take three parameters and create an Incident object, which is then added to our incidents list.

PYTHON
def record_incident(start_time, severity, description):
    incident = Incident(start_time, severity, description)
    incidents_list.append(incident)

# Example usage:
record_incident("2023-10-01 14:00:00", "critical", "Application error on server A")
Full Implementation

Combining all these steps together:

PYTHON
class Incident:
    def __init__(self, start_time, severity, description):
        self.start_time = start_time
        self.severity = severity
        self.description = description

incidents_list = []

def record_incident(start_time, severity, description):
    incident = Incident(start_time, severity, description)
    incidents_list.append(incident)

# Example usage:
record_incident("2023-10-01 14:00:00", "critical", "Application error on server A")

#### Triaging the Question

The question is asking for a Python function that records an incident with start time, severity, and description into a list. The interviewer might be probing your understanding of object-oriented programming concepts (like defining classes) as well as basic data structures.

#### Assumptions

Start times are provided as strings in ISO 8601 format.
Severity is a string such as "critical," "major," or "minor."
Description is a brief text summary.

#### Common Mistakes
and Follow-Ups
Mistake: Not defining the Incident class properly, leading to incorrect incident recording.

#### Follow-Up
Discussing how you would handle other metadata that might be relevant (e.g., end time, resolution time).
Mistake: Not appending the incident object correctly to the list.

#### Follow-Up
Extending the system to support logging and querying incidents.

