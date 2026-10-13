---
id: "TOOLS/Python/Python_Language/OpenTelemetry_Instrumentation"
level: 4
type: "concept"
topic: "TOOLS/Python/Python_Language"
tools: ["Python", "Prometheus", "AWS"]
---

# OpenTelemetry Instrumentation

**L4 Advanced** · [↑ Python Language](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Python](<../_index.md>)
- [Prometheus](<../../Prometheus/_index.md>)
- [AWS](<../../AWS/_index.md>)

## Practice questions

- [Basic] [What does it mean for a metric to breach an SLA, and how would you write Python code that flags any response time above a 500ms target in a list?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37636_What_does_it_mean_for_a_metric_to_breach_an_SLA_and_how_woul.md>)
- [Intermediate] [How would you get basic system metrics like CPU and memory usage in Python, and what does the psutil library provide?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-36977_How_would_you_get_basic_system_metrics_like_CPU_and_memory_u.md>)
- [Intermediate] [How would you use Python to measure the round-trip latency of an HTTP health check and store it as a metric sample?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37690_How_would_you_use_Python_to_measure_the_round-trip_latency_o.md>)
- [Intermediate] [How would you write a Python script that finds idle or oversized EC2 instances by correlating CloudWatch CPU metrics with instance types, to drive a rightsizing recommendation?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37045_How_would_you_write_a_Python_script_that_finds_idle_or_overs.md>)
- [Intermediate] [How would you set up structured logging and metrics in a Python model service so you can answer 'how many predictions per class, at what latency, with what feature-null rate' from a dashboard?](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37228_How_would_you_set_up_structured_logging_and_metrics_in_a_Pyt.md>)
- [Deep] [Explain how you'd instrument a Python automation service so that every AWS API call it makes is traced with OpenTelemetry, letting you answer why a deploy took 12 minutes.](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37042_Explain_how_you_d_instrument_a_Python_automation_service_so_.md>)
- [Deep] [Explain how you'd instrument your own Python AIOps service with OpenTelemetry so that when detection is slow, you can tell whether it's ingest, scoring, enrichment, or the notify call.](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37796_Explain_how_you_d_instrument_your_own_Python_AIOps_service_w.md>)
- [Deep] [Explain how you'd instrument a Python training pipeline to emit per-stage timing and resource metrics so you can attribute a slow run to data loading vs compute vs checkpointing.](<../../../PRACTICE/TOOLS/Python/Python_Language/Q-37244_Explain_how_you_d_instrument_a_Python_training_pipeline_to_e.md>)

## Related concepts

- [Prometheus TSDB and Scrape Health](<../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Prometheus_TSDB_and_Scrape_Health.md>) *(Time Series Databases)*
- [Tail Latency Investigation](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Tail_Latency_Investigation.md>) *(Cloud Fundamentals)*

