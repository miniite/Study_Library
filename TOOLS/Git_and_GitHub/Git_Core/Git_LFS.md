---
id: "TOOLS/Git_and_GitHub/Git_Core/Git_LFS"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub", "AWS"]
---

# Git LFS

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [AWS](<../../AWS/_index.md>)

## Practice questions

- [Basic] [How would you keep large experiment log files out of Git while still being able to share them with your team?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27133_How_would_you_keep_large_experiment_log_files_out_of_Git_whi.md>)
- [Basic] [Why would you keep a small sample of log lines in the repo but keep the full production log archive in external storage?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27564_Why_would_you_keep_a_small_sample_of_log_lines_in_the_repo_b.md>)
- [Intermediate] [What is Git LFS and why is it commonly used for storing model weights or images in an ML repository?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27078_What_is_Git_LFS_and_why_is_it_commonly_used_for_storing_mode.md>)
- [Intermediate] [Why is it a bad idea to commit a large local Parquet output or a DuckDB database file, and how would DVC or Git LFS help if you truly must track a sample?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27969_Why_is_it_a_bad_idea_to_commit_a_large_local_Parquet_output_.md>)
- [Intermediate] [A designer keeps committing large model-eval screenshots and sample audio into the GenAI repo, bloating it. How would you introduce Git LFS mid-project, migrate existing binaries, and what breaks for people without LFS installed?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27475_A_designer_keeps_committing_large_model-eval_screenshots_and.md>)
- [Intermediate] [A designer keeps committing large PNG and Figma exports that bloat the frontend repo. How would you introduce Git LFS mid-project, migrate the existing binaries, and what breaks for people who don't have LFS installed?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28211_A_designer_keeps_committing_large_PNG_and_Figma_exports_that.md>)
- [Deep] [What is Git LFS and why is it needed for large binary files?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28655_What_is_Git_LFS_and_why_is_it_needed_for_large_binary_files.md>)
- [Deep] [You track large model checkpoints with Git LFS, but a rollback to last week's tag pulls a checkpoint that LFS already garbage-collected from the remote. How would you couple LFS artifact retention to Git-referenced tags so any tagged model is always restorable?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27169_You_track_large_model_checkpoints_with_Git_LFS_but_a_rollbac.md>)

## Related concepts

- [Resilience to Git Outages](<../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>) *(GitOps)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Distributed Cache Design](<../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>) *(High Level Design)*

