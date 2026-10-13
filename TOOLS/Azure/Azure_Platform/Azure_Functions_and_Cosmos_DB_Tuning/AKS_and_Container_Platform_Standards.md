---
id: "TOOLS/Azure/Azure_Platform/AKS_and_Container_Platform_Standards"
level: 5
type: "deep-dive"
topic: "TOOLS/Azure/Azure_Platform"
tools: ["Azure", "Kubernetes"]
---

# AKS and Container Platform Standards

**L5 Expert** · [↑ Azure Platform](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Azure](<../../_index.md>)
- [Kubernetes](<../../../Kubernetes/_index.md>)

## Practice questions

- [Basic] [What is Azure Kubernetes Service (AKS) and what does Azure manage for you versus what you manage?](<../../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03924_What_is_Azure_Kubernetes_Service_AKS_and_what_does_Azure_man.md>)
- [Deep] [Your AKS cluster's kubelet-managed nodes keep going NotReady under load spikes; walk me through how you'd diagnose whether it's node pressure, PLEG, or the Azure CNI IP exhaustion.](<../../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03697_Your_AKS_cluster_s_kubelet-managed_nodes_keep_going_NotReady.md>)
- [Deep] [You need exactly-once-ish processing from Event Hubs into AKS consumers; how do checkpointing, partitions, and consumer groups affect ordering and replay?](<../../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03740_You_need_exactly-once-ish_processing_from_Event_Hubs_into_AK.md>)
- [Deep] [An AKS upgrade to a new Kubernetes version broke an admission webhook; how do you plan and stage cluster upgrades to catch API deprecations safely?](<../../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03767_An_AKS_upgrade_to_a_new_Kubernetes_version_broke_an_admissio.md>)
- [Deep] [How do you decide when a workload should leave AKS for Azure Container Apps or Functions to reduce operational burden, and how do you communicate that platform-tier decision to 50 teams?](<../../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03830_How_do_you_decide_when_a_workload_should_leave_AKS_for_Azure.md>)
- [Deep] [Explain how you'd assess and adopt Azure Container Apps' dapr and KEDA capabilities as an org standard versus keeping everything on AKS; what workloads move and what stays?](<../../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03885_Explain_how_you_d_assess_and_adopt_Azure_Container_Apps_dapr.md>)
- [Expert] [You need to enforce mTLS between AKS services; compare doing it with a service mesh (Istio/Linkerd/Open Service Mesh) versus app-level, and the operational cost.](<../../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03762_You_need_to_enforce_mTLS_between_AKS_services_compare_doing_.md>)
- [Expert] [How do you design a policy for AKS node pool consolidation and bin-packing across a fleet to reclaim 30% waste, and what reliability guardrails keep it from causing eviction storms?](<../../../../PRACTICE/TOOLS/Azure/Azure_Platform/Q-03844_How_do_you_design_a_policy_for_AKS_node_pool_consolidation_a.md>)

## Related concepts

- [Cost Allocation Data Pipelines](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Allocation_Data_Pipelines.md>) *(FinOps)*
- [Kafka Capacity and Cost](<../../../Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/Kafka_Capacity_and_Cost.md>) *(Kafka Core)*
- [Choosing an Event Backbone](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>) *(Distributed Systems Core)*

## References

- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Data factory support › Platform support capabilities
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — Repair scheduling Minimize the impact of repairs on your application by scheduling them at off-peak
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 7Scaling Data Solutions › CHAPTER 7Scaling Data Solutions(#contents.xhtmlch07a)
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running tasks in containers

