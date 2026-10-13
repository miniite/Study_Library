---
type: "practice"
difficulty: "Deep"
---

# You need a Python script that computes a diff between the desired and actual set of DNS records (via a provider API) and applies only the delta. How would you make it converge safely and avoid accidental mass deletion?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prompt Injection Defense](<../../../../TOOLS/Python/Python_Language/Prompt_Injection_Defense.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to develop a Python script that intelligently computes the difference between the desired DNS records (input by the user) and the actual DNS records fetched from a provider API. The goal is to apply only the necessary changes (the "delta") while ensuring safety and avoiding accidental mass deletions.

The core idea here is to implement a strategy where we first identify what needs to be added, updated, or removed based on the input and current state, and then apply these changes in a controlled manner. This involves fetching the current DNS records, comparing them with the desired ones, and executing a series of operations that only add or update existing records, rather than deleting anything.

#### Triage

The question is essentially asking for a robust script to manage DNS records by ensuring safe updates while minimizing risk. The primary concerns are:

Fetching the current state from an API.
Comparing desired and actual states to find differences.
Applying only the necessary changes (delta).
Ensuring safety, especially avoiding mass deletions.

We can assume the following:

We have a function fetch_dns_records() that retrieves the current DNS records via the provider API.
The input is a list of desired DNS records, each represented as a dictionary with keys like name, type, and content.
Step-by-Step Solution

Fetch Current State: Use the fetch_dns_records() function to get the current state of DNS records.

Compare Desired vs Actual: Compare the desired set of DNS records with the actual set by keying on unique identifiers (e.g., record names and types).

Apply Changes Safely:

For each desired record, check if it already exists in the current state.
If it exists, update its details if needed.
If not, add the new record.
For records that no longer exist in the desired set, mark them for deletion but only under controlled conditions.

Execute Changes: Apply changes one at a time, logging each action and ensuring all operations are reversible or idempotent.

#### Example
Code
PYTHON
import json

def fetch_dns_records():
    # Simulate fetching DNS records via API (for demonstration)
    return [
        {"name": "www.example.com", "type": "A", "content": "192.0.2.1"},
        {"name": "mail.example.com", "type": "MX", "priority": 10}
    ]

def desired_records():
    # Simulate user input of desired DNS records
    return [
        {"name": "www.example.com", "type": "A", "content": "192.0.2.1"},
        {"name": "mail.example.com", "type": "MX", "priority": 5}
    ]

def apply_dns_changes(current_records, desired_records):
    current_dict = {f"{r['name']}.{r['type']}": r for r in current_records}
    desired_dict = {f"{d['name']}.{d['type']}": d for d in desired_records}

    changes = {
        "additions": [],
        "updates": [],
        "deletions": []
    }

    # Identify additions and updates
    for key, record in desired_dict.items():
        if key not in current_dict:
            changes["additions"].append(record)
        elif set(record.keys()) - {"content"} != set(current_dict[key].keys()):
            changes["updates"].append({
                "current": {k: v for k, v in current_dict[key].items() if k in record},
                "desired": {k: v for k, v in record.items()}
            })

    # Identify deletions
    for key in current_dict.keys() - desired_dict.keys():
        changes["deletions"].append(current_dict[key])

    return changes

def main():
    current_records = fetch_dns_records()
    desired_records_data = json.dumps(desired_records(), indent=4)
    print("Desired DNS Records:")
    print(desired_records_data)

    changes = apply_dns_changes(current_records, desired_records())
    if not any(changes.values()):
        print("No changes needed.")
    else:
        for action, records in changes.items():
            if action == "additions":
                print(f"Adding {len(records)} new record(s):")
                for r in records:
                    print(json.dumps(r, indent=4))
            elif action == "updates":
                print(f"Updating {len(records)} existing record(s):")
                for update in records:
                    current = {k: v for k, v in update["current"].items()}
                    desired = {k: v for k, v in update["desired"].items()}
                    print(json.dumps(current, indent=4))
                    print(" -> ")
                    print(json.dumps(desired, indent=4))
            elif action == "deletions":
                print(f"Marking {len(records)} record(s) for deletion:")
                for r in records:
                    print(json.dumps(r, indent=4))

if __name__ == "__main__":
    main()

#### Closing Summary

The script ensures safe and controlled DNS record updates by first comparing the desired set with the current state, then applying only necessary changes. By avoiding deletions unless explicitly requested or necessary, we minimize risk of accidental mass deletion.

#### Common Mistakes/Follow-Ups
Mass Deletions: Always prioritize updating existing records over deleting them unless explicitly commanded.
Concurrency Issues: Handle potential race conditions by ensuring all API requests are idempotent and retryable.
Logging and Reversibility: Maintain a log of changes for auditing purposes, allowing easy reversal if something goes wrong.

