---
type: "practice"
difficulty: "Intermediate"
---

# How would you generate documentation automatically from annotated YAML config files so the config schema and docs never drift?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Linting YAML](<../../../../TOOLS/YAML/YAML_Core/Linting_YAML.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To automatically generate documentation from annotated YAML configuration files, we need to create a system that parses the YAML files, extracts relevant metadata (comments or annotations), and generates corresponding documentation in Markdown or HTML format. This ensures that the schema and documentation are always in sync.

#### Triage

The core of this question is about developing an automated pipeline that processes YAML config files and generates comprehensive documentation. The interviewer might be probing your understanding of YAML syntax, metadata extraction techniques, and integration with continuous integration (CI) systems. We will assume that you have access to the YAML files stored in a version control system like Git.

#### First-Principles Solution
YAML Parsing: Use a YAML parser to read and traverse the configuration files.
Metadata Extraction: Identify and extract annotations or comments used as metadata for documenting each section or field.
Template Generation: Use Jinja2 or similar templating engines to generate documentation based on the extracted metadata.
Automation & Integration: Integrate this process into a CI pipeline, so every time changes are committed or merged, the documentation is automatically updated.
Step-by-Step Solution

YAML Parsing:

Use a Python library like PyYAML to parse and traverse the YAML files.
PYTHON
import yaml

with open('config.yaml', 'r') as file:
    config = yaml.safe_load(file)

Metadata Extraction:

Define annotations or comments in YAML that indicate which sections need documentation. For example, add # doc: <description> at the beginning of each section.
YAML
service:
  # doc: A brief description of the service
  name: web
  version: 1.0
  ports:
    - # doc: HTTP port for the web application
      port: 80

Template Generation:

Use Jinja2 to generate documentation based on the metadata.
PYTHON
from jinja2 import Template

template = """
# Configuration Documentation
{% for section, values in config.items() %}
## {{ section }}

{%- for key, value in values.items() %}
- **{{ key }}**: {{ value }}
  {%- if 'doc' in value and value['doc'] %}
    ({{ value['doc'] }})
  {% endif %}
{%- endfor %}
{% endfor %}
"""

rendered_doc = Template(template).render(config=config)

Automation & Integration:

Integrate this process into a CI pipeline using tools like Jenkins, GitLab CI, or GitHub Actions.
Write a script that will run during the build phase to generate the documentation.

#### Closing Summary

By following these steps, we can ensure that our configuration files and their corresponding documentation are always aligned. This not only helps in maintaining clear and understandable configurations but also reduces the risk of bugs introduced due to misaligned schema and documentation.

#### Common Mistakes
& Follow-Ups
Missing Annotations: Ensure all relevant sections have proper annotations.
Template Syntax Errors: Double-check your Jinja2 templates for any syntax issues.
Integration Issues: Verify that the CI pipeline correctly triggers the documentation generation process after code changes.

#### Example
PYTHON
# Sample YAML file: config.yaml
---
service:
  # doc: A brief description of the service
  name: web
  version: 1.0
  ports:
    - # doc: HTTP port for the web application
      port: 80

# Jinja2 template in template.html
{{ render_template(config) }}

# Generated documentation output
# Configuration Documentation
## service
- **name**: web (A brief description of the service)
- **version**: 1.0
- **ports**:
  - **port**: 80 (HTTP port for the web application)

**Comparison:**

```
Section	Key	Value	Doc
service	name	web	A brief description of the service
service	version	1.0
service	ports	[80]	HTTP port for the web application
```

**Example input / output:**

```text
# Sample YAML file: config.yaml
---
service:
  # doc: A brief description of the service
  name: web
  version: 1.0
  ports:
    - # doc: HTTP port for the web application
      port: 80

# Jinja2 template in template.html
{{ render_template(config) }}

# Generated documentation output
# Configuration Documentation
## service
- **name**: web (A brief description of the service)
- **version**: 1.0
- **ports**:
    - **port**: 80 (HTTP port for the web application)
```

**Flow:**

1. Yes
2. Read YAML file
3. Parse YAML?
4. Extract Metadata
5. Generate Documentation
6. Integrate with CI

