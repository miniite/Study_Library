---
type: "practice"
difficulty: "Intermediate"
---

# Why is it helpful to pair your prompt-experiment notebook with a plain .py file using jupytext so Git diffs of your LLM experiments are readable?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Notebooks in Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Notebooks_in_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When conducting AI experiments, especially those involving large language models (LLMs), it's crucial to keep track of changes and iterations. Pairing a Jupyter notebook with a plain .py file via jupytext is beneficial because it allows you to maintain readable Git diffs while preserving the interactive nature of your experiments. This approach combines the benefits of both environments: notebooks for experimentation and documentation, and plain Python scripts for clean diffs.

#### Triage

The question is essentially asking why using jupytext to synchronize Jupyter notebooks with .py files can make it easier to manage changes in LLM experiments through Git diffs. The core idea is that jupytext ensures the diffs remain human-readable, making it simpler to understand what has changed between commits.

#### First Principles Solution
Jupyter Notebooks vs. Plain Python Files:
Jupyter notebooks are interactive and allow for easy experimentation, documentation, and visualization.
Plain .py files are straightforward and easy to read, but lack the interactivity of Jupyter notebooks.
Using jupytext:
jupytext allows you to write a single source file that can be either a Jupyter notebook or a Python script.
When committed to Git, this dual format ensures that diffs are readable and meaningful.
Step-by-Step Solution
Initial Setup:
Install jupytext using pip: pip install jupytext.
Configuration:
Configure your Jupyter notebook to use jupytext. For example, you can add a comment at the top of the notebook specifying that it should be synced with a .py file.
Committing Changes:
When you make changes in the notebook, commit them using Git.
The diff will show changes only in the relevant cells or lines, rather than the entire file.

#### Example

Let's say you have an LLM experiment setup where you're fine-tuning a model for text generation. You might want to test different hyperparameters and observe the results interactively in a Jupyter notebook. Here’s how jupytext can help:

Before:

PY
# plain.py
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained("distilgpt2")
tokenizer = AutoTokenizer.from_pretrained("distilgpt2")

def generate_text(prompt):
    inputs = tokenizer(prompt, return_tensors="pt")
    outputs = model.generate(inputs["input_ids"], max_length=50)
    return tokenizer.decode(outputs[0])

# Test
print(generate_text("Once upon a time"))

After:

MARKDOWN
# generate_text.ipynb
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained("distilgpt2")
tokenizer = AutoTokenizer.from_pretrained("distilgpt2")

def generate_text(prompt):
    inputs = tokenizer(prompt, return_tensors="pt")
    outputs = model.generate(inputs["input_ids"], max_length=50)
    return tokenizer.decode(outputs[0])

# Test
print(generate_text("Once upon a time"))
Git Diff:
When you commit the changes in generate_text.ipynb, Git will show diffs only for the cells that have been modified.
DIFF
@@ -1,5 +1,6 @@
 from transformers import AutoModelForCausalLM, AutoTokenizer

+def generate_text(prompt):
    model = AutoModelForCausalLM.from_pretrained("distilgpt2")
    tokenizer = AutoTokenizer.from_pretrained("distilgpt2")

  - def generate_text(prompt):
  + inputs = tokenizer(prompt, return_tensors="pt")
  + outputs = model.generate(inputs["input_ids"], max_length=50)

#### Closing Summary
and Common Mistakes

Using jupytext to pair Jupyter notebooks with plain .py files helps in maintaining readable diffs while keeping the interactive nature of experiments. Common mistakes include not properly configuring jupytext, leading to unexpected behavior during commits or pull requests.

Self-Check

Correctness + completeness confirmed.

