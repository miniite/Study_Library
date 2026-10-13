---
type: "practice"
difficulty: "Basic"
---

# Why do Jupyter notebooks produce messy Git diffs, and what makes them hard to review in a pull request?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Notebooks in Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Notebooks_in_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Jupyter notebooks can create messy Git diffs because they store both code and outputs, which often contain non-code elements like images or plots that are not relevant for version control purposes. This makes diffs hard to review in a pull request as they include unnecessary artifacts.

#### Triage
Question: Why do Jupyter notebook diffs produce messy diffs?
What is being probed: Understanding of Git principles, the structure of Jupyter notebooks, and best practices for managing code.

#### Assumptions
You are familiar with basic Git operations and understand that diffs are a way to track changes in files.
Step-by-Step Solution:
Understanding Jupyter Notebooks:
Jupyter notebooks (.ipynb) are interactive documents containing both code (Python, R, etc.) and rich media outputs like plots, images, and text.
Git’s Role in Version Control:
Git tracks changes by comparing file revisions but struggles with binary files or large outputs that are not structured as code.
File Structure of Jupyter Notebooks:
A .ipynb file is a JSON object containing:
JSON
{
  "cells": [...],
  "metadata": {...},
  "nbformat": 4,
  "nbformat_minor": 5
}
cells store the actual content (code, markdown, etc.).
Outputs and metadata are stored as part of the JSON structure.
Why Messy Diffs Occur:
When a cell runs in Jupyter, it generates outputs that may be binary data or large objects not easily parsed by Git.
Git diff commands often show these outputs (e.g., images) as changes, leading to diffs with many lines that are hard to interpret.

#### Closing Summary

Jupyter notebooks produce messy Git diffs because they store non-code elements like outputs and images within the JSON structure. This makes it difficult for reviewers to understand what actual code has changed without diving into the notebook interactively.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the context of Jupyter notebooks in Git diff reviews.

#### Follow-Up
How can we manage this issue effectively? Possible solutions include:
Manually removing outputs before committing.
Using tools like nbstripout to remove non-code elements.
Storing outputs separately or generating them on demand.

