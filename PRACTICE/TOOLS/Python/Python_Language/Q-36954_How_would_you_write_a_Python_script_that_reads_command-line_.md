---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python script that reads command-line arguments so an operator can pass an environment name and a version to deploy?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Subprocess Safety](<../../../../TOOLS/Python/Python_Language/Subprocess_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To create a Python script that reads command-line arguments, we need to understand how to parse these inputs and ensure our script is robust. Essentially, you want your operator to be able to pass an environment name (like "staging" or "production") and a version number (like "1.0.4") from the command line.

#### Triage

The question asks us to write a Python script that can accept two command-line arguments: an environment name and a deployment version. The interviewer is probing our understanding of:

How to handle command-line arguments in Python.
Error handling for incorrect inputs.
Using a structured approach to parse and validate the data.

#### First Principles Solution

We will use Python's argparse module, which makes it easy to handle command-line options. Here’s how we can do this step-by-step:

Import argparse: This is necessary to define and manage command-line arguments.
Define Argument Parser: Set up the parser with required and optional arguments.
Parse Arguments: Read the input from the command line.
Validation: Ensure that the inputs are valid before proceeding.
Python Script

Here’s a complete script:

PYTHON
import argparse

def main():
    # Define the argument parser
    parser = argparse.ArgumentParser(description="Deploy application to specified environment with version.")

    # Add required arguments
    parser.add_argument('--env', dest='environment', type=str, help="Environment name (e.g., staging)")
    parser.add_argument('--version', dest='version', type=str, help="Version number to deploy")

    # Parse the command-line arguments
    args = parser.parse_args()

    # Validate inputs
    if not args.environment or not args.version:
        print("Both environment and version are required.")
        return

    # Here you would implement your deployment logic using `args.environment` and `args.version`
    print(f"Deploying to {args.environment} with version {args.version}")

if __name__ == "__main__":
    main()

#### Closing Summary
+ Common Mistakes/Follow-Ups

This script uses the argparse module to handle command-line arguments, ensuring that both the environment name and version are provided. It also includes basic validation to check if these parameters are supplied.

#### Common Mistakes

Forgetting to validate input types.
Not handling cases where required arguments might be missing.
Additional Considerations

You might want to add more robust error handling or logging mechanisms, especially for production environments. Additionally, you could extend this script to handle different deployment strategies based on the environment specified (e.g., rollback procedure in staging vs. production).

