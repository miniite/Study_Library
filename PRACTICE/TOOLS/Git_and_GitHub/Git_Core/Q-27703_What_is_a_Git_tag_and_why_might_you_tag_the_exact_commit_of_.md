---
type: "practice"
difficulty: "Basic"
---

# What is a Git tag, and why might you tag the exact commit of the schema that shipped with your v1.0 release?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Tags and Releases](<../../../../TOOLS/Git_and_GitHub/Git_Core/Tags_and_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Git tags are used in version control systems like Git to mark specific commits as important or significant milestones. In the context of database development, tagging a specific commit is particularly useful when you want to ensure that a particular release can be easily reproduced later.

In this scenario, we're asked about tagging the exact commit for the schema used in the v1.0 release. This practice helps in several ways:

Ensuring reproducibility: Tagging allows us to go back and exactly replicate the state of our database schema at any point in time.
Reference points: Tags serve as reference points, making it easy to identify specific versions of your code or database schema.

Now let's explore this step-by-step from first principles:

Understanding Git Tags: A tag is essentially a pointer to a specific commit in the repository’s history. You can create a lightweight or annotated tag. Annotated tags are more useful as they include metadata like author, timestamp, and message, making them easier to track.

Why Tagging Matters for Database Schemas:

Reproducibility: When you release v1.0 of your application, tagging the exact schema commit ensures that if you need to go back and fix a bug or recover data, you can do so using the known good state.
Version Control: Tags help in managing multiple versions of your codebase or database schema. This is particularly useful when dealing with complex applications where different components might be at different stages of development.
Documentation: Tags act as documentation points, marking significant changes and milestones in the project's history.

Creating a Tag:

SH
git tag v1.0_schema "Version 1.0 database schema"

This command creates an annotated tag named v1.0_schema with a message describing it as the version 1.0 database schema. You can also use lightweight tags if you prefer, but for critical points like this, annotated tags are recommended.

Pushing Tags: To make sure everyone on your team or in production has access to this tag, you need to push it to the remote repository.

SH
git push origin v1.0_schema

Retrieving a Tagged Version: If you need to revert to the v1.0 schema for any reason, you can check out that tag.

SH
git checkout tags/v1.0_schema -b temp_branch

This command checks out the commit pointed to by v1.0_schema into a new branch named temp_branch.

#### Common Mistakes

Failing to push the tag to the remote repository can result in others not being able to access it.
Overlooking tagging can lead to inconsistencies when reproducing previous states of your schema.

