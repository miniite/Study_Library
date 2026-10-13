---
type: "practice"
difficulty: "Deep"
---

# A hotfix index needs to reach the current release branch, main, and two older LTS release branches whose schema diverged. Compare cherry-pick -x, rebase --onto, and format-patch/am for landing the migration cleanly on each and preserving provenance.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Cherry-picking and Backports](<../../../../TOOLS/Git_and_GitHub/Git_Core/Cherry-picking_and_Backports.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this scenario where a hotfix index needs to be applied across multiple release branches with varying schemas, we need to compare three strategies: cherry-pick -x, rebase --onto, and format-patch. Each method has its own trade-offs in terms of complexity, potential conflicts, and preservation of provenance. Let's break down each approach step by step.

**Comparison:**

```
Method	Description	Complexity	Conflicts Handling	Provenance Preservation
cherry-pick -x	Applies specific commits as new commits in the current branch.	Low to Medium	Requires careful handling of merge conflicts	Preserves original commit hashes and authors
rebase --onto	Rebases a range of commits onto a new base, effectively moving those commits to a different part of history.	High	Potential for complex conflict resolution	Preserves commit history but moves the commit log messages
format-patch	Creates patch files from a range of commits, which can be applied elsewhere using git am.	Medium	Allows manual inspection and editing of patches before application	Provides detailed patch information and allows review
```

**Code:**

```bash
# Example commands for the different methods
git cherry-pick -x <commit-hash>
git rebase --onto <new-base-branch> <old-commit> main
git format-patch <start-commit>..<end-commit>
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Cherry-Pick -x
8. Apply specific commits as newcommits in the current branch
9. Merge conflicts?
10. Handle merge conflictsmanually
11. Preserve original commithashes and authors, simpleprocess
12. Rebase --onto
13. Move a range of commits to adifferent part of history
14. Potential complex conflictresolution needed?
15. Complex rebase with potentialdata loss if not handledcarefully
16. Preserve commit history butmove log messages, highcomplexity
17. Format-Patch
18. Create patch files from a rangeof commits
19. Allow manual inspection andediting before applying?
20. Detailed patches with reviewcapabilities, mediumcomplexity
21. Preserve commit historythrough detailed patches butwith potential applicationissues if not reviewed carefully

