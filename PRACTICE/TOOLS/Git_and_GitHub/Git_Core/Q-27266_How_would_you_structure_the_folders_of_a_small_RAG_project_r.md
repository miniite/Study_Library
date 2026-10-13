---
type: "practice"
difficulty: "Basic"
---

# How would you structure the folders of a small RAG project repo so prompts, ingestion code, and app code are cleanly separated?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Sharing Projects on GitHub](<../../../../TOOLS/Git_and_GitHub/Git_Core/Sharing_Projects_on_GitHub.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To structure the folders of a small RAG (Retrieval-Augmented Generation) project repository such that prompts, ingestion code, and application code are cleanly separated, we need to create a clear and organized directory layout. This separation ensures maintainability, readability, and ease of collaboration among team members.

#### Triage

The question is essentially asking how to organize the file structure of a RAG project in Git/GitHub. The key aspects being probed include:

Modularization: How to separate different functionalities.
Best Practices: Adherence to common coding standards and conventions.
Organizational Structure: Ensuring that related files are grouped together logically.

#### First-Principles Solution

We will use a standard directory structure where each major component (prompts, ingestion code, app code) is placed in its own subdirectory. This approach promotes clarity and ease of navigation within the repository.

Project Root Directory: Contains all top-level files like README.md, .gitignore, etc.
Prompts Directory: Stores all prompt templates used by the RAG system.
Ingestion Code Directory: Houses scripts or programs responsible for collecting and preprocessing data to be used by the RAG system.
App Code Directory: Contains the main application logic that interacts with the RAG model, handles user requests, and generates responses.
Detailed Structure

Here's a sample structure:

CODE
my-rag-project/
├── README.md
├── .gitignore
├── prompts/
│   ├── prompt_template_1.json
│   ├── prompt_template_2.yaml
│   └── ...
├── ingestion/
│   ├── data_collection.py
│   ├── preprocess_data.py
│   └── ...
└── app/
    ├── main.py
    ├── config.py
    └── models/
        ├── rag_model.py
        └── ...

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key to a well-structured repository is clear separation of concerns. By organizing the files into distinct directories, we ensure that each part of the application has its dedicated space. Common mistakes include:

Lack of Structure: Keeping everything in one directory can lead to disorganized code.
Overcomplication: Creating unnecessary subdirectories for minor components.

In follow-ups, you might be asked about version control practices or how to manage dependencies within these directories.

**Code:**

```
MERMAID
Copy
graph LR;
    A[Project Root] --> B[README.md]
    A --> C[.gitignore]
    A --> D[prompts]
    A --> E[ingestion]
    A --> F[app]
```

**Flow:**

1. Project Root
2. README.md
3. .gitignore
4. prompts
5. ingestion
6. app

