---
type: "practice"
difficulty: "Basic"
---

# What entries would you add to a .gitignore file in a Python LLM app repo that uses a .env for API keys and a local Chroma vector store?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Gitignore](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitignore.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working on a Python LLM (Language Model) application, it's essential to ensure sensitive information such as API keys is not committed to version control. A .gitignore file helps manage which files and directories should be ignored by Git. For an LLM app that uses a .env for storing API keys and a local Chroma vector store, you'll need to add specific entries in your .gitignore to exclude these sensitive files.

The question is really about understanding how to protect your application's secrets while still being able to develop and collaborate effectively. The core idea here is to ensure that the .env file and the local database directory are not accidentally committed, but you can still work on them locally without issues.

Let's break this down from first principles:

API Key Management: You have a .env file containing API keys for your LLM service.
Local Data Storage: You use a Chroma vector store for local data storage in the application.

Here’s how you would structure your .gitignore file:

PLAINTEXT
# .env file - contains sensitive information like API keys
.env

# Local Chroma vector store directory - this is where the local database files are stored
local_chroma_store/

By adding these entries, Git will ignore both the .env file and the local_chroma_store/ directory when committing changes. This ensures that your sensitive data remains secure.

