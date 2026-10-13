---
type: "practice"
difficulty: "Basic"
---

# How would you keep large experiment log files out of Git while still being able to share them with your team?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git LFS](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_LFS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To manage large experiment log files in a way that keeps them out of Git but allows sharing among team members, we can use a combination of GitHub's features and external storage solutions.

#### Triaging the Question

The core challenge here is to store significant file sizes (e.g., logs from experiments) without including them directly in Git repositories. This avoids bloating the repository and makes it easier to manage and share changes with teammates. The key points the interviewer might be probing include understanding how to leverage GitHub's storage capabilities, considering alternative solutions for storing large files, and ensuring that these mechanisms facilitate seamless collaboration.

#### Solution
Use GitHub Releases or Packages:
GitHub provides the ability to upload binary assets like logs through releases or packages. These can be shared with teammates via download links.
GitHub Actions Workflows:
Automate the process of creating and uploading these log files by integrating them into your CI/CD pipeline using GitHub Actions.
Step-by-Step Solution:
Create a Release for Large Files:
Use git tag to mark specific versions of your project.
Utilize GitHub Releases API or GitHub CLI to upload large files (e.g., logs) as part of the tagged release.
GitHub Actions Workflow Example:
YAML
name: Log Upload

on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize]

jobs:
  log-uploader:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0  # This ensures the entire history is available for tagging.

      - name: Setup Python
        uses: actions/setup-python@v4

      - name: Install dependencies
        run: |
          pip install requests

      - name: Upload Logs
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          RELEASE_NAME: "Logs v1.0"
          LOG_PATH: "path/to/your/logs/*.log"
        run: |
          echo ::set-output name=release_url::
            $(curl --location --request POST 'https://api.github.com/repos/your-username/your-repo/releases' \
              --header 'Authorization: token $GITHUB_TOKEN' \
              --header 'Content-Type: application/json' \
              --data-raw '{
                "tag_name": "${{ env.RELEASE_NAME }}",
                "target_commitish": "main"
              }' | jq -r '.upload_url')

          # Upload log files
          curl --location --request POST "$release_url" \
            --header 'Authorization: token $GITHUB_TOKEN' \
            --form "log_file=@$LOG_PATH;type=text/plain"
Documentation and Communication:
Ensure that the team is aware of how to access these logs via GitHub Releases or Packages.
Document the workflow for uploading and downloading files.

#### Closing Summary

By using GitHub's release feature, we can keep large experiment log files outside our main Git repository while still maintaining version control and sharing capabilities. This approach ensures that our Git history remains lightweight and easy to manage.

#### Common Mistakes/Follow-Ups

Mistake: Overlooking the size limit of direct file uploads in Git repositories.

#### Follow-Up
Discuss GitHub's size limits and alternative methods for handling very large files (e.g., using AWS S3 or Google Cloud Storage).

Mistake: Not integrating this into your CI/CD pipeline, leading to manual and error-prone processes.

#### Follow-Up
Highlight the importance of automating the process with GitHub Actions.

