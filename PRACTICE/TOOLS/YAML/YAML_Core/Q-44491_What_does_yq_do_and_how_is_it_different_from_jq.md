---
type: "practice"
difficulty: "Intermediate"
---

# What does 'yq' do and how is it different from 'jq'?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ YAML and JSON](<../../../../TOOLS/YAML/YAML_Core/YAML_and_JSON.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To quickly understand yq and its distinction from jq, let’s first summarize their core functionalities, then dive into what each tool does and why you might use one over the other.

TL;DR:

jq is primarily used for parsing and manipulating JSON data.
yq extends jq to handle YAML files, making it a versatile tool for processing both JSON and YAML configurations in DevOps workflows.

#### Triage

The question asks about the differences between yq and jq. Both are command-line tools used for transforming structured data. However, jq is specifically designed for JSON, while yq serves as an extension that supports YAML files along with JSON.

#### Assumptions
The reader has basic knowledge of DevOps tooling and common file formats like JSON and YAML.
We are comparing jq and yq in the context of working with configuration files in a CI/CD pipeline or managing infrastructure as code (IaC).

#### First Principles Solution

Core Functionality:

Both jq and yq leverage the same underlying principles for filtering, transforming, and querying structured data.

JSON Handling (jq):

jq is a powerful tool for processing JSON documents. It offers extensive functionality through its command-line interface and can be used to extract, manipulate, and transform data based on various criteria using JavaScript-like syntax.

YAML Support (yq):

yq builds upon the capabilities of jq by extending it to support YAML files. This makes it particularly useful for scenarios where your configuration or data is stored in YAML format.

Use Cases:

For JSON-based configurations, you would use jq.
For YAML-based configurations, you would use yq.
Step-by-Step Reasoning

Understanding jq:

jq processes JSON inputs and outputs structured data in a way that is easy to understand and manipulate.

#### Example
Using jq to filter and transform JSON data.
SH
cat config.json | jq '.settings | {theme, fonts}'

Introducing yq:

yq adds YAML support by extending the functionality of jq.
It enables the same powerful filtering and transformation capabilities on YAML files as on JSON.

#### Example
Using yq to manipulate a YAML file.
SH
cat config.yaml | yq '.settings.theme'

Differences in Syntax and Usage:

The syntax for both tools is very similar, but yq provides additional commands specific to YAML, such as handling anchors and aliases.

#### Example
of a complex query using yq.
SH
cat inventory.yaml | yq 'with_select(.machines[].name | contains("dev"))'

#### Closing Summary
+ Common Mistakes/Follow-Ups
Both tools are powerful but serve different file formats. jq is indispensable for JSON, while yq broadens the scope to YAML.
A common mistake is using jq on a YAML file or vice versa without recognizing the format differences.

#### Follow-up
questions might include specific scenarios where one tool would be more appropriate than the other.

**Code:**

```
SH
Copy
# Example of jq usage with JSON
 cat config.json | jq '.settings | {theme, fonts}'

 # Example of yq usage with YAML
 cat config.yaml | yq '.settings.theme'
```

