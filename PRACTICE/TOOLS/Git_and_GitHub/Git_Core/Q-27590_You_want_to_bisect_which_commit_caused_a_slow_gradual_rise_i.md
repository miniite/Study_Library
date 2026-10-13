---
type: "practice"
difficulty: "Expert"
---

# You want to bisect which commit caused a slow, gradual rise in alert-noise volume over two weeks, but each candidate requires replaying a window of telemetry through the correlation engine. How would you adapt git bisect run to a replay-and-measure loop, and what makes bisect converge on the wrong commit here?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Git Bisect](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Bisect.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to leverage git bisect run while adapting it to incorporate a custom measurement loop that simulates replaying telemetry through our correlation engine. Git's bisect tool is designed for binary search in version control history to identify the first bad commit; however, its standard implementation doesn't directly support complex side effects like replaying and measuring telemetry data.

TL;DR

We will adapt git bisect run by customizing it to execute a measurement loop that replays telemetry through our correlation engine. This involves creating a script for the measurement process and using git bisect run with this script, ensuring we handle potential false positives carefully.

#### Triaging the Question

The core of the question revolves around efficiently identifying the first bad commit in a large codebase where each candidate requires a complex operation (replaying telemetry through a correlation engine) to determine its validity. Git's bisect tool provides a framework for binary search, but we need to integrate it with our specific use case.

#### Assumptions
and First-Principles
Telemetry Data: We have historical telemetry data covering the period of interest.
Correlation Engine: This engine processes telemetry and generates alerts based on predefined rules.
Git Repository Structure: The repository contains multiple commits, each representing a state change in code that might affect alert volume.
Step-by-Step Solution

Identify Commit Range:

Define the range of commits (e.g., last two weeks) where the slow rise began.

Create Measurement Script:

BASH
# measure-telemetry.sh
# $1 is the commit hash to test

git checkout $1  # Checkout the specified commit
python3 ./correlation_engine.py replay-timeline <historical-telemetry-file> > replayed-data.json

alerts=$(cat replayed-data.json | jq '.alerts[].count' | paste -sd+ - | bc)  # Sum up alert counts
echo $alerts

Configure Git Bisect:

BASH
git bisect start <start-commit> <end-commit>

# Use `git bisect run` with the measurement script
git bisect run measure-telemetry.sh

Handling Edge Cases and False Positives:

Potential false positives occur if a commit changes telemetry data slightly, but not enough to affect alert volume.
Implement heuristics in the measurement script to filter out commits that show minimal impact.

#### Closing Summary
and Common Mistakes/Follow-Ups

By adapting git bisect run with a custom measurement loop, we ensure efficient identification of the first bad commit. Key challenges include handling false positives and managing large amounts of telemetry data. Potential follow-ups could involve optimizing the correlation engine to reduce computation time or implementing parallelization for faster testing.

#### Common Mistakes
False Positives: Failing to account for minimal changes in telemetry data that might not significantly affect alert volume.
Performance Overhead: Running the full telemetry replay for each commit can be resource-intensive, potentially slowing down the entire process.

