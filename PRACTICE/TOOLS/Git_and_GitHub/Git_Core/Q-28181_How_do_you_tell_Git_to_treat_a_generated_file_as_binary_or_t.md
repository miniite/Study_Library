---
type: "practice"
difficulty: "Intermediate"
---

# How do you tell Git to treat a generated file as binary or to always take one side on merge via gitattributes?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Gitattributes and Merge Drivers](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitattributes_and_Merge_Drivers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git, it's crucial to handle certain files correctly—especially those that are not text-based or where you want to avoid unnecessary conflicts during merges. This answer will cover how to tell Git to treat a generated file as binary and how to configure merge strategies via gitattributes.

#### Triage

The question is about configuring Git attributes, specifically for handling binary files and managing merge strategies to ensure consistent behavior when these files are involved in a repository.

Key Concepts
Git Attributes: Git allows you to specify per-file rules using .gitattributes files. These rules can tell Git how to handle different types of files.
Merge Strategies: You can configure Git to use specific merge strategies for certain file patterns, ensuring that conflicts are handled in a predictable manner.

#### First Principles Solution
Identify the File Type:
For binary files (e.g., image files, executable binaries), you want to tell Git not to attempt a text-based comparison.
Configure .gitattributes:
Create or edit your repository's .gitattributes file to specify that certain patterns should be treated as binary.
Step-by-Step Solution

Create/Edit .gitattributes File

PLAINTEXT
# Open the .gitattributes file in a text editor

Add Binary Pattern:

Add a line to mark files as binary:
PLAINTEXT
*.jpg filter=lfs diff=lfsmime-type=image/jpeg
*.exe diff=binary eol=crlf
The filter and diff options are used for large files (like those managed by Git LFS). Here, we use the binary option to treat these files as binary.

Configure Merge Strategy

To always take one side on a merge (say, always keep the local version), you can configure the merge strategy:
PLAINTEXT
[merge "always"]
    name = Always Take Local Version
    driver = true

Apply Changes:

After adding these lines to .gitattributes, commit the changes.

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Not specifying patterns correctly can lead to unintended file handling.
Forgetting to add or commit the .gitattributes file.
Misconfiguring merge strategies, leading to unexpected behavior during merges.

#### Follow-Ups

Exploring other Git attributes like ident, crlf, and lf.
Understanding how different merge drivers work (e.g., binary, ours, theirs).

#### Example
of .gitattributes File
PLAINTEXT
# Always treat .jpg files as binary
*.jpg filter=binary

# Always take the local version during a merge for *.exe files
[merge "local-exe"]
    name = Local Exe Strategy
    driver = true

# Large file management (example with Git LFS)
*.* filter=lfs diff=lfsmime-type=image/jpeg

#### Conclusion

By carefully configuring your .gitattributes and merge strategies, you can ensure that Git handles specific files as binary and merges them in a consistent manner. This is particularly useful for managing large or non-text-based files.

**Code:**

```plaintext
# Example .gitattributes file content
*.jpg filter=binary
*.exe diff=binary eol=crlf

[merge "local-exe"]
    name = Local Exe Strategy
    driver = true
```

