---
id: "TOOLS/Python/Python_Language/Drift-triggered_Retraining"
level: 5
type: "deep-dive"
topic: "TOOLS/Python/Python_Language"
tools: ["Python"]
---

# Drift-triggered Retraining

**L5 Expert** · [↑ Python Language](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../../_index.md>)

## Practice questions

- [Intermediate] [How do you set and use a random seed with np.random so that an experiment is reproducible, and why does reproducibility matter in ML?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37135_How_do_you_set_and_use_a_random_seed_with_np_random_so_that_.md>)
- [Deep] [Your Python log-clustering detector treats every new deploy's log format change as a wave of anomalies. How would you make it robust to benign template churn while still catching real new error signatures?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37806_Your_Python_log-clustering_detector_treats_every_new_deploy_.md>)
- [Deep] [How would you write a Python function to compute a stratified time-based split for a time-series model so you never train on the future, and why does naive shuffling break temporal models?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37197_How_would_you_write_a_Python_function_to_compute_a_stratifie.md>)
- [Deep] [Explain how you'd reduce the artifact size of a trained model in Python (pruning, quantization, removing training-only state) so deployment and cold starts are cheaper without hurting accuracy much.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37255_Explain_how_you_d_reduce_the_artifact_size_of_a_trained_mode.md>)
- [Deep] [Explain how you'd build an org-wide Python golden-file/snapshot testing standard so a feature-engineering change can't silently alter existing features for any model that depends on it.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37316_Explain_how_you_d_build_an_org-wide_Python_golden-file_snaps.md>)
- [Deep] [A principal probe: your org keeps hitting float32-vs-float64 drift between pandas preprocessing and the trained model. How would you enforce a consistent-precision contract across the whole Python scoring path platform-wide, and detect violations in CI?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37352_A_principal_probe_your_org_keeps_hitting_float32-vs-float64_.md>)
- [Expert] [Explain how you'd make a Python training job deterministic end-to-end (data order, weight init, dropout) so two runs with the same seed produce identical models, and why full determinism is hard on GPU.](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37195_Explain_how_you_d_make_a_Python_training_job_deterministic_e.md>)
- [Expert] [Your fine-tuned model started producing worse outputs than the base model on some inputs. How would you use a Python eval set to detect regression and decide whether the fine-tune, the data, or the eval is at fault?](<../../../../PRACTICE/TOOLS/Python/Python_Language/Q-37489_Your_fine-tuned_model_started_producing_worse_outputs_than_t.md>)

## Related concepts

- [Shared Pipeline Platforms](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/Shared_Pipeline_Platforms.md>) *(Pipeline Orchestration)*
- [Reproducible Training Data Versions](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Reproducible_Training_Data_Versions.md>) *(Data Architecture)*
- [Performance Regression Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Performance_Regression_Gates.md>) *(CI CD)*

