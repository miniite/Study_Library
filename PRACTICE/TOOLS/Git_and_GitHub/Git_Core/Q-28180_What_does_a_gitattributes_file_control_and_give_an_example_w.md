---
type: "practice"
difficulty: "Intermediate"
---

# What does a .gitattributes file control, and give an example where it changes diff or merge behavior?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Gitattributes and Merge Drivers](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitattributes_and_Merge_Drivers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A .gitattributes file in Git is used to specify per-file attributes such as how files should be filtered by Git (e.g., text vs binary) and how they should be handled during diff, merge, or checkout operations. In essence, it gives you fine-grained control over how Git treats individual files. Let’s dive into the core concept first before exploring an example.

**Code:**

```plaintext
# Example .gitattributes file
*.log filter=catEOL eol=crlf
```

**Flow:**

1. Developer 1 modifies app.log
2. Appends a newline to ensureconsistency with catEOL filter
3. Commits changes
4. Pushes to remote repositorywith LF line endings
5. Developer 2 pulls from samerepo
6. CRLF line endings in local .logfiles are expected
7. Git detects differences duringmerge, but uses catEOL filterto normalize lines before diffand merge operations
8. Merge proceeds smoothly withnormalized line endings

