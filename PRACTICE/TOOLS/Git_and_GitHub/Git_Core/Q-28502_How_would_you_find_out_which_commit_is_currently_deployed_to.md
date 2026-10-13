---
type: "practice"
difficulty: "Intermediate"
---

# How would you find out which commit is currently deployed to production if all you have is the running container?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reproducible Builds from Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reproducible_Builds_from_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine which commit is currently deployed in a production environment, given only a running container, we need to leverage Git and GitHub. This involves understanding how containers are tagged, deployed, and versioned, as well as using tools like git and Docker commands effectively.

#### Triage

The question primarily tests your ability to:

Understand the relationship between container tags and Git commits.
Use Docker commands to inspect running containers.
Retrieve relevant commit information from GitHub or a similar source control repository.

#### Assumptions
include:

You have access to the production environment where the container is running.
The container tag is associated with a specific commit in your version control system (e.g., Git).

#### Solution

Identify Container Tag: Start by identifying the Docker image tag used for the production deployment. This can often be found in deployment scripts, Kubernetes manifests, or environment variables.

Fetch Image Details:

BASH
docker inspect <container_name_or_id> | grep "Image"

Use this command to get the full image name and tag of the running container.

Extract Tag from Image Name: The output will provide a Docker image URL, typically in the format repository:tag. Extract the tag part using tools like awk.

BASH
docker inspect <container_name_or_id> | grep "Image" | awk -F':' '{print $2}'

Identify Commit: Use the Git repository to find out which commit corresponds to this tag. If you're familiar with how your deployment pipeline works, you might have a script or a tool that maps tags to commits.

Manual Mapping (if needed):

Navigate to your GitHub repository and look for tags.
For each tag, use git log to find the commit ID associated with it.
BASH
git show <tag_name>
Compare this output with the commit logs in your local Git repository or on GitHub.

Cross-Verification: Verify that the commit you found matches by checking its content and comparing it with a known snapshot from your development environment.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By following these steps, we can determine which commit is currently deployed to production. Common mistakes include not knowing how to extract the tag information correctly or assuming the tag-to-commit mapping is not available. Always ensure you have access to the necessary tools and repositories.

For further exploration:

Automate this process using scripts in your deployment pipeline.
Ensure that tags are consistently and meaningfully named for easy identification.

#### Cons
ider integrating a CI/CD tool like Jenkins, GitLab CI, or GitHub Actions to automate deployments and tag management.

**Code:**

```bash
docker inspect <container_name_or_id> | grep "Image" | awk -F':' '{print $2}'
```

