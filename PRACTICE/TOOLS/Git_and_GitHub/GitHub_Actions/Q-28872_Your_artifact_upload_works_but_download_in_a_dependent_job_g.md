---
type: "practice"
difficulty: "Intermediate"
---

# Your artifact upload works but download in a dependent job gets an empty directory. What are the common causes and fixes?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pinning Third-party Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Pinning_Third-party_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using GitHub Actions, if you have an artifact upload that works properly but a subsequent job fails to download it as expected (receiving an empty directory), several issues could be at play. This problem often arises due to misconfigurations in the actions/upload-artifact and actions/download-artifact steps or because of environmental factors like caching, permissions, or workflow sequence.

#### Triage

The core issue here is that while the artifact upload step is successful, the download step isn't completing as intended. We need to ensure that:

The correct name and path are used for the artifact.
Permissions and access tokens are correctly set up in both steps.
No caching or other actions inadvertently interfere with the file system.

#### First-Principles Solution

Let’s break this down step-by-step:

Step 1: Verify Artifact Name and Path

Ensure that the name attribute is consistent between the upload and download steps. The name must match exactly, including case sensitivity.

YAML
# Example of artifact setup
jobs:
  upload-artifact-job:
    runs-on: ubuntu-latest
    steps:
      - name: Upload Artifact
        uses: actions/upload-artifact@v3
        with:
          name: my-artifact
          path: /path/to/my/artifact

  download-artifact-job:
    needs: [upload-artifact-job]
    runs-on: ubuntu-latest
    steps:
      - name: Download Artifact
        uses: actions/download-artifact@v3
        with:
          name: my-artifact  # Ensure this matches the upload step exactly!
Step 2: Check Permissions and Tokens

Ensure that your workflow has the necessary permissions to access the artifact. The actions/download-artifact step should be part of a job or a subsequent job in the same workflow run.

YAML
# Example with proper permissions
jobs:
  upload-and-download:
    runs-on: ubuntu-latest
    steps:
      - name: Upload Artifact
        uses: actions/upload-artifact@v3
        with:
          name: my-artifact
          path: /path/to/my/artifact

      - name: Download Artifact
        uses: actions/download-artifact@v3
        with:
          name: my-artifact
Step 3: Address Caching Issues

Sometimes, caching can interfere with the file system. Ensure that your workflow doesn't accidentally cache files or directories in a way that would cause conflicts.

YAML
# Example of disabling unnecessary caches
jobs:
  upload-and-download:
    runs-on: ubuntu-latest
    steps:
      - name: Upload Artifact
        uses: actions/upload-artifact@v3
        with:
          name: my-artifact
          path: /path/to/my/artifact

      - name: Download Artifact
        uses: actions/download-artifact@v3
        with:
          name: my-artifact

      # Disable caching if it's causing issues
      - run: echo "cache.disable=true" >> $GITHUB_OUTPUT
Step 4: Debugging and Logs

If the issue persists, enable detailed logging in your GitHub Actions. This can help you identify where things are going wrong.

YAML
# Enable debug logs
jobs:
  upload-and-download:
    runs-on: ubuntu-latest
    steps:
      - name: Upload Artifact
        uses: actions/upload-artifact@v3
        with:
          name: my-artifact
          path: /path/to/my/artifact
        env:
          ACTIONS_STEP_DEBUG: true

      - name: Download Artifact
        uses: actions/download-artifact@v3
        with:
          name: my-artifact
        env:
          ACTIONS_STEP_DEBUG: true

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key to resolving this issue is ensuring consistency in artifact names, correct permissions, and addressing any caching or environmental factors. If you still encounter issues after these steps, consider checking for other common mistakes such as file path issues, token expiration, or race conditions between jobs.

#### Common mistakes
include:

Mismatched artifact names.
Misconfigured workflow permissions leading to unauthorized access.
Ignoring the impact of caching mechanisms on the file system.

For further investigation, you can also explore GitHub's official documentation and community forums for additional troubleshooting tips.

