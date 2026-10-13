---
id: "TOOLS/Git_and_GitHub/Git_Core/Versioning_Data_with_DVC"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub"]
---

# Versioning Data with DVC

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)

## Practice questions

- [Intermediate] [What is DVC (Data Version Control) and how does it complement Git for machine learning projects?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27064_What_is_DVC_Data_Version_Control_and_how_does_it_complement_.md>)
- [Deep] [When you track a dataset with DVC, what actually gets committed to Git versus what is stored in remote storage like S3?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27065_When_you_track_a_dataset_with_DVC_what_actually_gets_committ.md>)
- [Deep] [How can DVC metrics and plots files, tracked in Git, let a reviewer see how a model's accuracy changed in a pull request?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27098_How_can_DVC_metrics_and_plots_files_tracked_in_Git_let_a_rev.md>)
- [Deep] [How does dvc repro decide which stages to rerun, and why does it depend on Git-tracked dependencies and outputs?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27113_How_does_dvc_repro_decide_which_stages_to_rerun_and_why_does.md>)
- [Deep] [Why might you tag each dataset snapshot in DVC and Git so you can always retrain on exactly the data used before?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27135_Why_might_you_tag_each_dataset_snapshot_in_DVC_and_Git_so_yo.md>)
- [Deep] [How would you explain the overall Git-plus-DVC-plus-MLflow workflow you'd use to keep an ML project reproducible from data to deployed model?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27154_How_would_you_explain_the_overall_Git-plus-DVC-plus-MLflow_w.md>)
- [Deep] [A retrain job in CI produced a model that regressed offline AUC by two points, and you suspect a silent change in a Git-tracked feature-store definition. Walk through using git log -L on the feature spec plus dvc diff to isolate whether it was the schema or the encoder.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27160_A_retrain_job_in_CI_produced_a_model_that_regressed_offline_.md>)
- [Deep] [You need the DVC data version and the Git code version to move as one unit so a rollback restores both, but a teammate committed code without committing the updated .dvc file. How would you enforce that data and code pins always advance together in a single commit?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27172_You_need_the_DVC_data_version_and_the_Git_code_version_to_mo.md>)

## Related concepts

- [Build and Release Metadata](<../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/_index.md>) *(CI CD)*
- [ML Model Delivery Pipelines](<../../../10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines.md>) *(CI CD)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*

