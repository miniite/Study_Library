---
id: "11_AI_and_ML/Applied_ML/Anomaly_Detection/Smoothing_and_Sensitivity_Trade-offs"
level: 4
type: "concept"
topic: "11_AI_and_ML/Applied_ML/Anomaly_Detection"
tools: []
---

# Smoothing and Sensitivity Trade-offs

**L4 Advanced** · [↑ Anomaly Detection](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a moving average and how can it help smooth a noisy time-series before anomaly detection?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Anomaly_Detection/Q-01823_What_is_a_moving_average_and_how_can_it_help_smooth_a_noisy_.md>)
- [Basic] [What is a percentile (like p95 or p99) and why is it useful for anomaly detection on latency?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Anomaly_Detection/Q-01850_What_is_a_percentile_like_p95_or_p99_and_why_is_it_useful_fo.md>)
- [Intermediate] [Why can watching only the average (mean) latency hide a real anomaly that percentiles reveal?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Anomaly_Detection/Q-01851_Why_can_watching_only_the_average_mean_latency_hide_a_real_a.md>)
- [Intermediate] [Why is it useful to alert on the rate of change (derivative) of a metric rather than its value?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Anomaly_Detection/Q-01868_Why_is_it_useful_to_alert_on_the_rate_of_change_derivative_o.md>)
- [Intermediate] [Why can watching p99 latency reveal an anomaly the mean hides, and how do you detect on percentiles at scale?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Anomaly_Detection/Q-01944_Why_can_watching_p99_latency_reveal_an_anomaly_the_mean_hide.md>)
- [Intermediate] [Explain the trade-off between smoothing a metric to reduce noise and losing short sharp spikes you need to catch.](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Anomaly_Detection/Q-01947_Explain_the_trade-off_between_smoothing_a_metric_to_reduce_n.md>)
- [Intermediate] [Why would you alert on the rate of change of a metric, and what noise problems does the derivative introduce?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Anomaly_Detection/Q-01966_Why_would_you_alert_on_the_rate_of_change_of_a_metric_and_wh.md>)
- [Deep] [What is exponential smoothing and how does it weight recent data more than old data?](<../../../PRACTICE/11_AI_and_ML/Applied_ML/Anomaly_Detection/Q-01847_What_is_exponential_smoothing_and_how_does_it_weight_recent_.md>)

## Related concepts

- [Scaling with Contaminated Data](<../../ML_Foundations/Data_Preprocessing/Scaling_with_Contaminated_Data.md>) *(Data Preprocessing)*
- [Distribution Change Alerts](<../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>) *(Data Observability)*
- [Detecting Unit Changes](<../../../02_Data_Engineering/Data_Quality/Validation/Detecting_Unit_Changes.md>) *(Validation)*

