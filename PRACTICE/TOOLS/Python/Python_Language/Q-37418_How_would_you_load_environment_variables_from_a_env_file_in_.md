---
type: "practice"
difficulty: "Basic"
---

# How would you load environment variables from a .env file in Python for storing your LLM API keys, and which library helps?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/Python_Language/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To load environment variables from a .env file in Python, we need to use a library that can parse and set these values as environment variables. One popular choice is python-dotenv, which makes it straightforward to manage configuration settings like LLM API keys.

#### Triaging the Question

The question asks about loading environment variables from a .env file specifically for storing LLM (Large Language Model) API keys. The core issue here involves managing secrets and ensuring they are securely stored and accessed within an application. We need to understand which library (python-dotenv) is commonly used for this purpose.

#### Assumptions
You have a .env file with your LLM API key.
Your Python script will run in the same environment as where you set the .env file.
You want to securely access these keys without hardcoding them in your source code.

#### First-Principles Solution

To load environment variables from a .env file, we can use the python-dotenv library. Here’s how it works:

Install python-dotenv: This can be installed via pip.
Create or Use an Existing .env File: The .env file should contain your LLM API key in the format API_KEY=your_key_here.
Load the .env File in Your Python Script using load_dotenv().
Steps to Implement
Install python-dotenv:
BASH
pip install python-dotenv
Create or use an existing .env file, for example:
ENV
API_KEY=your_llm_api_key_here
In your Python script, load the environment variables from the .env file at the start of your script:
PYTHON
# Import required libraries
from dotenv import load_dotenv
import os

# Load the .env file
load_dotenv()

# Access the API key using os.getenv()
api_key = os.getenv('API_KEY')

print(f"Loaded LLM API Key: {api_key}")

#### Closing Summary
and Common Mistakes/Follow-Ups

Using python-dotenv is a common practice to manage environment variables securely. Always ensure that your .env file is stored outside of your version control system, such as Git, to avoid exposing sensitive information.

#### Common Mistakes

Not installing the required library (pip install python-dotenv).
Forgetting to call load_dotenv() at the beginning of the script.
Hardcoding API keys directly into the source code.

#### Example

Let’s walk through an example:

Install and Import Libraries:
BASH
pip install python-dotenv
Create a .env File in your project directory:
ENV
API_KEY=your_llm_api_key_here
Load the Environment Variables in Your Python Script:
PYTHON
from dotenv import load_dotenv
import os

# Load the .env file
load_dotenv()

# Access the API key using os.getenv()
api_key = os.getenv('API_KEY')

print(f"Loaded LLM API Key: {api_key}")
Self-Check

This solution is correct and complete.

**Code:**

```python
from dotenv import load_dotenv
import os

# Load the .env file
load_dotenv()

# Access the API key using os.getenv()
api_key = os.getenv('API_KEY')

print(f"Loaded LLM API Key: {api_key}")
```

**Example input / output:**

```

pip install python-dotenv

Create a .env file with:

API_KEY=your_llm_api_key_here

Python script example:

from dotenv import load_dotenv import os

load_dotenv() api_key = os.getenv('API_KEY') print(f"Loaded LLM API Key: {api_key}")

CODE
Copy
Output:
```bash
Loaded LLM API Key: your_llm_api_key_here
```

