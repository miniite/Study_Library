---
id: "07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Scaling_for_Traffic_Spikes"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/APIs/API_Gateway"
tools: ["Kubernetes", "AWS"]
---

# Gateway Scaling for Traffic Spikes

**L5 Expert** · [↑ API Gateway](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Kubernetes](<../../../../TOOLS/Kubernetes/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [Your API gateway pods on Kubernetes keep getting OOMKilled under a traffic spike while CPU is fine; how do you size memory requests/limits for a proxy that buffers request bodies, and what metrics tell you it's buffering not leaking?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02781_Your_API_gateway_pods_on_Kubernetes_keep_getting_OOMKilled_u.md>)
- [Deep] [Your gateway autoscales on CPU but a spike of TLS handshakes saturates it before CPU crosses the HPA threshold; what custom metric would you scale on and how do you avoid flapping?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02803_Your_gateway_autoscales_on_CPU_but_a_spike_of_TLS_handshakes.md>)
- [Deep] [Your gateway's p99 is fine but p999 spikes every few minutes correlated with GC or config reload; how do you separate proxy-internal jitter from upstream jitter, and what do you tune first?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02834_Your_gateway_s_p99_is_fine_but_p999_spikes_every_few_minutes.md>)
- [Deep] [Your gateway's route table has grown to 12000 entries and cold-start config parsing now takes 8 seconds per node; how do you restructure route storage and matching so reloads stay fast and memory bounded?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02447_Your_gateway_s_route_table_has_grown_to_12000_entries_and_co.md>)
- [Deep] [How do you handle large file uploads and downloads through a gateway without buffering entire payloads in memory; what streaming and body-size settings matter?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02711_How_do_you_handle_large_file_uploads_and_downloads_through_a.md>)
- [Expert] [A gateway plugin calls an external policy service (OPA) synchronously per request and adds 15ms; how do you cache decisions, batch, or push policy to the gateway to cut this without letting stale policy authorize revoked access?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02477_A_gateway_plugin_calls_an_external_policy_service_OPA_synchr.md>)
- [Expert] [Describe how connection pooling and keep-alive between the gateway and upstreams affect latency and failure modes; what happens when pools are too small or too large?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02800_Describe_how_connection_pooling_and_keep-alive_between_the_g.md>)
- [Expert] [How would you load test a gateway to find its real ceiling; what metrics tell you it is the gateway (not upstreams) that saturates, and how do you find the limiting resource?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02915_How_would_you_load_test_a_gateway_to_find_its_real_ceiling_w.md>)

## Related concepts

- [Serverless Limits and Throttling](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>) *(Cloud Fundamentals)*
- [API Rate Limiting](<../../REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*
- [Org-wide Backpressure](<../../../System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*

