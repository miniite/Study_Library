---
type: "practice"
difficulty: "Deep"
---

# What is the difference between include:local include:project and include:remote?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Cache vs Artifacts](<../../../../TOOLS/GitLab/GitLab_CI/Cache_vs_Artifacts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing GitLab CI, we often talk about how to reuse configurations across different pipelines. The include keyword in GitLab CI allows us to reference other .gitlab-ci.yml files within our repository or from remote repositories. However, there are subtle differences between include:local, include:project, and include:remote.

Let's break down the core solution first:

TL;DR
Local (include:local): Refers to a file within the same project.
Project (include:project): Refers to a file from another project in your GitLab instance.
Remote (include:remote): Specifies a specific URL for an external repository.

#### Triage
and Assumptions

The question is asking us to understand the nuances of using include in different contexts within GitLab CI. We need to consider the following:

The structure of a GitLab project.
How files are referenced within the same or other repositories.
The implications on pipeline execution.

#### First Principles Solution
1. Local (include:local)

When you use include:local, it means that you are referencing a file from another stage or job in the same .gitlab-ci.yml file of your current project. Here's an example:

YAML
stages:
  - build
  - test

build_job:
  stage: build
  script: echo "Building"

include:
  local: other_stage/another_build_job.yml # This references another job or stage within the same .gitlab-ci.yml file.
2. Project (include:project)

include:project is used to include a file from another project in your GitLab instance. You can do this by specifying the project's namespace and ID:

YAML
stages:
  - build
  - test

build_job:
  stage: build
  script: echo "Building"

include:
  project: group/project_name#12345 # Replace with actual namespace and ID of the target project.
  file: other_stage/another_build_job.yml
3. Remote (include:remote)

include:remote is used to include a file from an external GitLab repository specified by its URL. This is useful for sharing configurations across multiple projects or environments:

YAML
stages:
  - build
  - test

build_job:
  stage: build
  script: echo "Building"

include:
  remote: https://gitlab.example.com/group/project_name.git#12345 # Replace with actual URL and commit/sha.
  file: other_stage/another_build_job.yml

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, the key differences are:

Local: Used within a single project to reference files from other stages/jobs in the same .gitlab-ci.yml file.
Project: Used to include files from another project within your GitLab instance.
Remote: Used to include files from an external repository.

#### Common mistakes
include:

Confusing local and project when referencing a different stage/job or project.
Failing to properly specify the URL in remote.
Not understanding that file must be specified when using project or remote.

In follow-up questions, you might be asked about best practices for configuration reuse, performance considerations, or how these include directives interact with GitLab's caching mechanisms.

**Code:**

```yaml
# Example of local include
stages:
  - build
  - test

build_job:
  stage: build
  script: echo "Building"

include:
  local: other_stage/another_build_job.yml # This references another job or stage within the same .gitlab-ci.yml file.

# Example of project include
stages:
  - build
  - test

build_job:
  stage: build
  script: echo "Building"

include:
  project: group/project_name#12345 # Replace with actual namespace and ID of the target project.
  file: other_stage/another_build_job.yml

# Example of remote include
stages:
  - build
  - test

build_job:
  stage: build
  script: echo "Building"

include:
  remote: https://gitlab.example.com/group/project_name.git#12345 # Replace with actual URL and commit/sha.
  file: other_stage/another_build_job.yml
```

