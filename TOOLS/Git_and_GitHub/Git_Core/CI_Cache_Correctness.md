---
id: "TOOLS/Git_and_GitHub/Git_Core/CI_Cache_Correctness"
level: 4
type: "concept"
topic: "TOOLS/Git_and_GitHub/Git_Core"
tools: ["Git_and_GitHub", "Python"]
---

# CI Cache Correctness

**L4 Advanced** · [↑ Git Core](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Git and GitHub](<../_index.md>)
- [Python](<../../Python/_index.md>)

## Practice questions

- [Intermediate] [What is caching in GitHub Actions and how does actions/cache speed up dependency installs?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28166_What_is_caching_in_GitHub_Actions_and_how_does_actions_cache.md>)
- [Intermediate] [How would you set up a GitHub Actions cache keyed on requirements.txt so your telemetry-analysis test workflow installs dependencies faster?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27555_How_would_you_set_up_a_GitHub_Actions_cache_keyed_on_require.md>)
- [Intermediate] [How would you cache Git dependencies or a cloned repo between CI runs to speed up builds?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-28479_How_would_you_cache_Git_dependencies_or_a_cloned_repo_betwee.md>)
- [Intermediate] [Your CI runs a training smoke test on every PR but it clones a 12 GB dataset via DVC each time, blowing past the runner timeout. How would you cache DVC-pulled data keyed on the .dvc lock so only changed data re-downloads?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27170_Your_CI_runs_a_training_smoke_test_on_every_PR_but_it_clones.md>)
- [Intermediate] [Your CI trains a telemetry-anomaly model on every PR by pulling a 12 GB metrics dataset via DVC each time, blowing the runner timeout. How would you cache DVC-pulled telemetry keyed on the .dvc lock so only changed windows re-download, and where does that cache go stale?](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27609_Your_CI_trains_a_telemetry-anomaly_model_on_every_PR_by_pull.md>)
- [Deep] [A GitHub Actions cache serving a stale database image gave a green migration run that failed against the real prod schema version. Design a cache-key strategy hashing the migration set and base-schema version that is correct yet fast, and how you'd detect a poisoned cache.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27836_A_GitHub_Actions_cache_serving_a_stale_database_image_gave_a.md>)
- [Deep] [Your GitHub Actions cache serves a stale conda environment, giving a green training run that fails in the serving container with a version mismatch. Design a cache-key strategy hashing the lockfile that's correct yet fast, and how you'd detect a poisoned cache.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27185_Your_GitHub_Actions_cache_serves_a_stale_conda_environment_g.md>)
- [Deep] [Explain why you'd pin third-party Actions in your GenAI CI to a full commit SHA instead of a tag like v4, what attack a mutable tag enables, and how you keep pinned SHAs updated without losing the guarantee.](<../../../PRACTICE/TOOLS/Git_and_GitHub/Git_Core/Q-27414_Explain_why_you_d_pin_third-party_Actions_in_your_GenAI_CI_t.md>)

## Related concepts

- [CI Caching](<../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Caching.md>) *(CI CD)*
- [Pinning Third-party Actions](<../GitHub_Actions/Pinning_Third-party_Actions.md>) *(GitHub Actions)*
- [Cache Invalidation](<../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/_index.md>) *(Caching)*

