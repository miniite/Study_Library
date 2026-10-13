---
type: "practice"
difficulty: "Deep"
---

# How would you enforce a data contract between an upstream producer and your Python pipeline so a breaking schema change is caught in CI rather than in production?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Handling Schema Evolution in Python](<../../../../TOOLS/Python/Python_Language/Handling_Schema_Evolution_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a breaking schema change in the data produced by an upstream system is caught during CI (Continuous Integration) rather than in production, we need to enforce a strict data contract between the producer and consumer systems. This can be achieved using Python by leveraging type annotations and a validation layer. Let’s break down the steps from first principles.

First, understand that a data contract defines the expected structure of the input data. In this case, we need to ensure that any changes in the schema (such as adding or removing fields) are caught early during CI rather than leading to runtime errors in production.

Secondly, we will use Python’s pydantic library for validation and type annotations to enforce this contract. Pydantic is a powerful tool that allows us to define models with strong typing, making it easy to validate incoming data against predefined schemas.

#### Triaging the Question

The question asks how to enforce a data contract so that any breaking changes are caught in CI rather than in production. The interviewer likely wants to see your understanding of:

Schema validation: How to ensure data conforms to an expected structure.
Type annotations and Pydantic models: Using Python’s pydantic library for robust schema validation.
Integration with CI/CD pipelines: Ensuring that the validation step is part of the build process.
Step-by-Step Solution

Define the Data Model: We first define a data model using Pydantic to specify the expected structure of the incoming data. This involves defining classes with type annotations and validators.

Implement Validation Logic: Use Pydantic’s validation logic to enforce the contract during runtime. If the input data does not conform to the defined schema, an exception will be raised.

Integrate with CI/CD Pipeline: Ensure that the validation step is part of the CI process. This can be done by adding a step in your CI pipeline (e.g., using GitHub Actions or Jenkins) that runs the data processing code and validates the input data before proceeding to further stages.

#### Example
Implementation

Here’s how we would implement this solution:

PYTHON
from pydantic import BaseModel, Field, validator

class DataModel(BaseModel):
    id: int
    name: str
    value: float = 0.0

    # Custom validators can be added here to enforce specific rules.
    @validator('value')
    def value_must_be_positive(cls, v):
        if v < 0:
            raise ValueError("Value must be positive")
        return v

def process_data(data: DataModel):
    print(f"Processing data with ID {data.id} and name {data.name}")

    # Your processing logic goes here
Integrating with CI/CD Pipeline

To integrate this validation step into a CI pipeline, you would add a GitHub Action or Jenkins job that runs the above code as part of the build process.

YAML
# Example GitHub Actions file
name: Data Validation

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  validate-data:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Python environment
      uses: actions/setup-python@v2
      with:
        python-version: '3.8'

    - name: Install dependencies
      run: |
        pip install pydantic

    - name: Run data validation
      run: |
        python -c "from your_module import DataModel, process_data; d = DataModel(id=1, name='test', value=-1); try: process_data(d) except Exception as e: print(e)"

#### Closing Summary
and Common Mistakes

In summary, enforcing a data contract in Python involves defining robust models using Pydantic, implementing validation logic, and integrating this step into the CI pipeline. Common mistakes to avoid include not covering edge cases (e.g., invalid values), neglecting to run validations in production, and failing to update tests when schema changes occur.

Always consider the broader context of your system and the potential impact of any changes. Regularly updating and validating models is crucial for maintaining data integrity throughout the lifecycle.

