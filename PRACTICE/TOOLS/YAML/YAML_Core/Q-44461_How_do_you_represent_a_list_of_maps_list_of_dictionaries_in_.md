---
type: "practice"
difficulty: "Intermediate"
---

# How do you represent a list of maps (list of dictionaries) in YAML? Give an example structure.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ YAML and JSON](<../../../../TOOLS/YAML/YAML_Core/YAML_and_JSON.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To represent a list of maps (or dictionaries) in YAML, we use square brackets [] to denote a list and indentation to define each map within that list. Each key-value pair is separated by a colon, with values indented under the corresponding key.

#### Triage

The question asks for how to structure a list of maps/dictionaries in YAML.
It's important to understand what a map (dictionary) looks like in YAML and how multiple such maps are grouped into a list.
We will provide an example that includes both a single map and a list of multiple maps.

#### First-principles solution

Single Map Example: A simple map with key-value pairs is represented as follows:

YAML
- name: Alice
  age: 25
  occupation: Developer

List of Maps Example: To represent a list of maps, each map is a separate entity in the list, and all are enclosed within square brackets []:

YAML
people:
  - name: Alice
    age: 25
    occupation: Developer
  - name: Bob
    age: 30
    occupation: Manager
  - name: Carol
    age: 28
    occupation: Designer

#### Closing summary
+ common mistakes/follow-ups:

Ensure that each map is properly indented under the list.
Pay attention to spacing and indentation, as YAML is sensitive to these details.
Use consistent indentation (usually two spaces) for key-value pairs.

#### Common mistakes
include not using square brackets [] to denote a list or incorrectly nesting maps within other structures without proper indentation. Additionally, forgetting to define the list name properly can lead to confusion in later processing steps.

**Code:**

```yaml
people:
  - name: Alice
    age: 25
    occupation: Developer
  - name: Bob
    age: 30
    occupation: Manager
  - name: Carol
    age: 28
    occupation: Designer
```

