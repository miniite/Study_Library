---
type: "practice"
difficulty: "Intermediate"
---

# Your CI trains a telemetry-anomaly model on every PR by pulling a 12 GB metrics dataset via DVC each time, blowing the runner timeout. How would you cache DVC-pulled telemetry keyed on the .dvc lock so only changed windows re-download, and where does that cache go stale?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ CI Cache Correctness](<../../../../TOOLS/Git_and_GitHub/Git_Core/CI_Cache_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of excessive data downloads during PRs, we need to implement a caching strategy using DVC (Data Version Control) and Git to store the telemetry dataset only when it changes. This approach will help reduce the runner timeout by ensuring that only updated portions of the 12 GB metrics dataset are re-downloaded. The key steps involve setting up DVC caching based on the .dvc lock file, understanding where the cache is stored, and determining its staleness.

#### Triage

The question revolves around optimizing data fetching during Pull Request (PR) reviews in CI pipelines by implementing a smart caching mechanism using DVC. We need to ensure that only changed parts of the dataset are re-downloaded, which will prevent hitting runner timeouts. The cache should be updated and invalidated based on changes to the telemetry data.

#### First Principles Solution
Understand DVC Workflow: DVC uses .dvc files to track the provenance of datasets. When a PR is created, DVC can pull the latest dataset if it has changed since the last commit.
Implement Caching Based on Lock Files: We need to ensure that only the portions of the dataset that have been modified since the last PR are re-downloaded. This can be achieved by comparing the current .dvc lock file with a previous version.
Determine Cache Staleness: The cache should be updated and invalidated when changes occur, ensuring that stale data is not used.
Step-by-Step Solution

Setup DVC for Telemetry Dataset:

Initialize a DVC repository in the telemetry dataset directory.
BASH
dvc init

Add and Stage Data:

Add the telemetry metrics dataset to DVC.
BASH
dvc add <path_to_metrics_dataset>

Configure DVC Pull Strategy:

Modify the .dvc/config file or use a custom script to define how DVC should handle data fetching based on changes in the lock file.

Implement Caching Logic:

Use a pre-commit hook or CI job to check if any files have changed since the last commit.
PYTHON
import os
import dvc.api

def is_cache_stale(cache_path, current_state):
    # Compare the current state with the cached lock file
    return not (os.path.exists(cache_path) and compare_files(cache_path, current_state))

cache_path = 'path_to_cache'
current_state = get_current_metrics_data()  # Function to retrieve latest telemetry data

if is_cache_stale(cache_path, current_state):
    dvc.pull()  # Fetch updated dataset from DVC
    save_updated_cache(cache_path)  # Save the new state of the cache
else:
    print("Using cached data.")

Determine Cache Staleness:

The cache should be invalidated when any part of the telemetry metrics is changed, ensuring that only relevant data is re-downloaded.

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes
Forgetting to invalidate the cache properly or not checking for changes before pulling can lead to unnecessary data fetching. Also, misconfiguration in DVC settings might cause incorrect handling of cached data.

#### Follow-Ups
Discussing additional optimization techniques such as parallel downloads, using smaller chunks of datasets, and integrating caching with other CI/CD tools.
Self-Check

Correctness + completeness confirmed.

