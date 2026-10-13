---
id: "TOOLS/Python/Python_Language/Flapping_Detection"
level: 4
type: "concept"
topic: "TOOLS/Python/Python_Language"
tools: ["Python"]
---

# Flapping Detection

**L4 Advanced** · [↑ Python Language](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../_index.md>)

## Practice questions

- [Basic] [What is an alert threshold, and how would you write a simple Python if-statement that prints an alert when CPU usage crosses 90 percent?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37634_What_is_an_alert_threshold_and_how_would_you_write_a_simple_.md>)
- [Intermediate] [How would you write a Python script that reads CPU readings and prints a warning only after three consecutive high values, to avoid single-spike alerts?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37658_How_would_you_write_a_Python_script_that_reads_CPU_readings_.md>)
- [Intermediate] [How would you write a Python script that reads a metric stream and computes an alert only when the value stays above a threshold for a sustained duration?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37711_How_would_you_write_a_Python_script_that_reads_a_metric_stre.md>)
- [Intermediate] [How would you implement a Python flapping-alert detector that identifies a metric oscillating across a threshold and suppresses it, while still surfacing a persistent breach?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37747_How_would_you_implement_a_Python_flapping-alert_detector_tha.md>)
- [Deep] [What is seasonality in metrics (like daily traffic peaks), and why does a naive fixed threshold in Python cause false alerts during expected busy hours?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37673_What_is_seasonality_in_metrics_like_daily_traffic_peaks_and_.md>)
- [Deep] [Your Python metric-ingestion service must handle late-arriving and out-of-order data points. How would you window and buffer them so a detector doesn't false-alarm on a temporarily incomplete series?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37752_Your_Python_metric-ingestion_service_must_handle_late-arrivi.md>)
- [Deep] [Your Python detector's threshold config is edited by many teams and a bad edit caused an alert blackout. How would you validate config changes and stage them so a bad threshold can't silence a whole service?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37770_Your_Python_detector_s_threshold_config_is_edited_by_many_te.md>)
- [Deep] [How would you write a Python test harness that replays a recorded incident's telemetry through your detector to verify it would have fired at the right time and not before?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37789_How_would_you_write_a_Python_test_harness_that_replays_a_rec.md>)

## Related concepts

- [Composite Monitors](<../../Datadog/Datadog_Platform/Composite_Monitors.md>) *(Datadog Platform)*
- [Distribution Change Alerts](<../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>) *(Data Observability)*
- [Dynamic vs Static Alerting](<../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Dynamic_vs_Static_Alerting.md>) *(Anomaly Detection)*

