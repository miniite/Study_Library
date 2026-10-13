---
type: "practice"
difficulty: "Intermediate"
---

# Your CI runs a training smoke test on every PR but it clones a 12 GB dataset via DVC each time, blowing past the runner timeout. How would you cache DVC-pulled data keyed on the .dvc lock so only changed data re-downloads?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ CI Cache Correctness](<../../../../TOOLS/Git_and_GitHub/Git_Core/CI_Cache_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of downloading a 12 GB dataset every time a PR is pushed, we need to implement caching strategies that leverage DVC's capabilities while ensuring minimal downtime during CI runs. The goal is to cache the dataset based on its .dvc lock file so that only changed data re-downloads.

The question revolves around understanding how DVC handles versioning and caching of datasets, and how this can be leveraged within a CI/CD pipeline to optimize data handling. Key assumptions include:

The CI runner is using dvc pull to fetch the dataset.
There's an existing .dvc file that describes the dataset and its provenance.

#### First Principles Solution

Understand DVC Caching Mechanism:

DVC uses Git under the hood for version control, but it also has a powerful caching mechanism that can be leveraged to store remote datasets locally.

Implement Cache Keys:

By default, DVC stores metadata about each dataset in its .dvc files and uses these as cache keys. However, we need to ensure that only changed data is re-downloaded.

Integrate with CI/CD Pipeline:

Modify the CI script to check if a cached version of the dataset exists before pulling from remote storage.

Cache Validation on PRs:

Use DVC's dvc status and dvc pull commands in combination with Git LFS to validate if the data has changed.
Detailed Steps

Pre-check with CI Script:

BASH
# Check for cached dataset
dvc status | grep 'OUTDATED' > /dev/null
if [ $? -eq 0 ]; then
  echo "Cached dataset is outdated, pulling new version..."
  dvc pull
else
  echo "Using cached dataset."
fi

Cache Management:

BASH
# In the CI script, ensure DVC cache is managed properly.
dvc fetch  # Fetch remote data if needed
dvc pull  # Pull data from remote storage if necessary

Validate Cache Integrity on PRs:

Run dvc status to check for any outdated files and only re-download if necessary.

#### Closing Summary

By implementing a caching strategy that checks the .dvc lock file, we can significantly reduce the CI runner timeout issues caused by frequent dataset downloads. This approach ensures that only changed data is re-downloaded, optimizing resource usage and CI/CD performance.

#### Common Mistakes/Follow-Ups

Failing to Validate Cache: Not running dvc status before pulling data.
Not Handling Edge Cases: Ignoring scenarios where the local cache might be corrupted or outdated.
Overwriting Changes Manually: Failing to automatically handle changes detected by DVC's tracking mechanisms.

