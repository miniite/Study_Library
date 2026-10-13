---
type: "practice"
difficulty: "Deep"
---

# A workflow needs to publish a release, tag it, and update a changelog atomically. How do you avoid partial-release states on failure?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pinning Third-party Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Pinning_Third-party_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your GitHub Actions workflow publishes a release, tags it, and updates a changelog atomically, avoiding partial releases in case of failures, we need to implement an atomic operation strategy using save-state and restore-state. This approach ensures that all steps either complete successfully or none do. Let's break down the solution step by step.

#### Triaging the Question

The core of this question is about ensuring atomicity in a multi-step GitHub Actions workflow, specifically when publishing a release. The key here is to ensure that either all steps complete successfully (release published, tagged, and changelog updated), or none do if any step fails.

#### First Principles Solution

To achieve this, we can use save-state and restore-state actions provided by GitHub Actions. These allow us to save the state of variables between jobs in a workflow, which we can then restore before proceeding with subsequent steps. If an error occurs during one of the steps, the job will fail, and because states are not persisted across jobs, no partial state changes will be committed.

Save State: Save the current state (e.g., whether a release is published or tagged) to a variable.
Fail Gracefully: If any step fails, mark the state as failed before proceeding with subsequent steps.
Restore State: Use the saved state to determine if it's safe to proceed.
Detailed Steps

Define Variables:

YAML
- name: Define Variables
  id: define_vars
  run: |
    echo "::set-output name=release_state::false"
    echo "::set-output name=tag_state::false"
    echo "::set-output name=changelog_state::false"
  save-state:
    release_state: ${{ steps.define_vars.outputs.release_state }}
    tag_state: ${{ steps.define_vars.outputs.tag_state }}
    changelog_state: ${{ steps.define_vars.outputs.changelog_state }}

Publish Release:

YAML
- name: Publish Release
  if: always()
  id: publish_release
  run: |
    echo "::set-output name=release_state::true"
    # Command to publish the release
  save-state:
    release_state: ${{ steps.publish_release.outputs.release_state }}

Tag Repository:

YAML
- name: Tag Release
  if: always()
  id: tag_repo
  run: |
    echo "::set-output name=tag_state::true"
    # Command to tag the repository
  save-state:
    tag_state: ${{ steps.tag_repo.outputs.tag_state }}

Update Changelog:

YAML
- name: Update Changelog
  if: always()
  id: update_changelog
  run: |
    echo "::set-output name=changelog_state::true"
    # Command to update the changelog
  save-state:
    changelog_state: ${{ steps.update_changelog.outputs.changelog_state }}

Final Check and Publish:

YAML
- name: Final Check
  if: always()
  run: |
    release_finalized=0
    tag_finalized=0
    if [ "$release_state" == "true" ] && [ "$tag_state" == "true" ] && [ "$changelog_state" == "true" ]; then
      echo "::set-output name=success::true"
      release_finalized=1
    else
      echo "::set-output name=success::false"
      tag_finalized=1
    fi

  - name: Publish Final State
    if: ${{ steps.final_check.outputs.success == 'true' }}
    run: |
      # Command to publish the final state of release, tag, and changelog

#### Closing Summary

By using save-state and restore-state, we ensure that each step in our workflow is atomic. If any step fails, it will not proceed to the next steps, leaving the system in a consistent state. This approach guarantees no partial releases or updates.

#### Common Mistakes/Follow-Ups
Not Using save-state: Failing to save and restore states can lead to inconsistent workflows.
Failing to Gracefully Fail: Not handling failures gracefully may result in unwanted changes being committed.
Complex Logic in Conditional Steps: Complex conditional logic can be hard to maintain; simpler is better.

**Code:**

```yaml
# Example GitHub Actions Workflow
on:
  push:
    tags:
      - 'v*'
jobs:
  publish-release:
    runs-on: ubuntu-latest
    steps:
      - name: Define Variables
        id: define_vars
        run: |
          echo "::set-output name=release_state::false"
          echo "::set-output name=tag_state::false"
          echo "::set-output name=changelog_state::false"
        save-state:
          release_state: ${{ steps.define_vars.outputs.release_state }}
          tag_state: ${{ steps.define_vars.outputs.tag_state }}
          changelog_state: ${{ steps.define_vars.outputs.changelog_state }}

      - name: Publish Release
        if: always()
        id: publish_release
        run: |
          echo "::set-output name=release_state::true"
          # Command to publish the release
        save-state:
          release_state: ${{ steps.publish_release.outputs.release_state }}

      - name: Tag Release
        if: always()
        id: tag_repo
        run: |
          echo "::set-output name=tag_state::true"
          # Command to tag the repository
        save-state:
          tag_state: ${{ steps.tag_repo.outputs.tag_state }}

      - name: Update Changelog
        if: always()
        id: update_changelog
        run: |
          echo "::set-output name=changelog_state::true"
          # Command to update the changelog
        save-state:
          changelog_state: ${{ steps.update_changelog.outputs.changelog_state }}

      - name: Final Check
        if: always()
        run: |
          release_finalized=0
          tag_finalized=0
          if [ "$release_state" == "true" ] && [ "$tag_state" == "true" ] && [ "$changelog_state" == "true" ]; then
            echo "::set-output name=success::true"
            release_finalized=1
          else
            echo "::set-output name=success::false"
            tag_finalized=1
          fi

        - name: Publish Final State
          if: ${{ steps.final_check.outputs.success == 'true' }}
          run: |
            # Command to publish the final state of release, tag, and changelog
```

