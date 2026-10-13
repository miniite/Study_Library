---
type: "practice"
difficulty: "Intermediate"
---

# You have a docker-compose.yml with a bad indentation error. How would you go about finding and fixing it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Indentation Pitfalls](<../../../../TOOLS/YAML/YAML_Core/Indentation_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a docker-compose.yml file that has an indentation issue, the key is to understand that YAML syntax relies heavily on proper indentation to define the structure of your services and configurations. Properly formatted files ensure consistent container definitions and prevent runtime errors.

#### Triage

The interviewer is asking how you would systematically identify and correct indentation issues in a docker-compose.yml file. This involves understanding the basic syntax rules, leveraging tools for validation, and making necessary corrections.

#### Assumptions
You are familiar with the structure of a typical docker-compose.yml file.
The issue is specifically related to indentation errors (e.g., incorrect spaces or tabs).

#### Solution
Step-by-Step Approach

Inspect the YAML Syntax

Understand that in YAML, indentation defines the hierarchy of nested structures. Common issues include inconsistent use of spaces and tabs.

Use a Linter Tool

Tools like yamllint or online validators can help you catch syntax errors before they become problematic.

#### Example
command to run yamllint:
SH
yamllint docker-compose.yml

Manually Review the File

Open the docker-compose.yml file in an editor that supports YAML syntax highlighting (e.g., VS Code, Sublime Text).
Check for nested structures like services within networks and volumes.

Correct Indentation Issues

Ensure consistent use of spaces or tabs across your file.
Common fixes include:
YAML
version: '3'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    networks:
      - my-network

networks:
  my-network:
    external: true

Validate the Corrected File

Run your Docker containers to ensure that the file is correctly interpreted.

#### Example
command:
SH
docker-compose up --build
If any errors occur, review and correct them.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, identifying and fixing indentation issues in a docker-compose.yml involves using linters to catch syntax errors, manually reviewing the file for consistency, making corrections, and validating the changes. Common mistakes include inconsistent use of spaces versus tabs or misusing indentation levels.

#### Follow-Up
Questions:

How do you handle more complex nested structures?
What steps would you take if the docker-compose.yml is part of a larger CI/CD pipeline?

**Code:**

```yaml
# Example of properly indented docker-compose.yml
version: '3'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    networks:
      - my-network

networks:
  my-network:
    external: true
```

