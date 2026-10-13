---
id: "07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Responsibilities_and_Anti-patterns"
level: 5
type: "deep-dive"
topic: "07_Distributed_Systems_and_Design/APIs/API_Gateway"
tools: ["Redis", "AWS"]
---

# Gateway Responsibilities and Anti-patterns

**L5 Expert** · [↑ API Gateway](<../_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../../TOOLS/Redis/_index.md>)
- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [How would you set up automatic cert renewal (cert-manager/ACME) for the gateway's public TLS, and what's your alerting so you never wake up to an expired cert 500ing every request?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02788_How_would_you_set_up_automatic_cert_renewal_cert-manager_ACM.md>)
- [Deep] [Your gateway needs to talk to upstreams over mTLS with certs rotated by a service mesh; how do you keep the gateway's client certs fresh, and what breaks during the rotation overlap window?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02805_Your_gateway_needs_to_talk_to_upstreams_over_mTLS_with_certs.md>)
- [Deep] [How do you roll out a stricter TLS floor (drop TLS 1.1) at the gateway with per-client compatibility telemetry, staged enforcement, and a measurable path to 100% without breaking a long tail of old clients?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02870_How_do_you_roll_out_a_stricter_TLS_floor_drop_TLS_1_1_at_the.md>)
- [Deep] [A gateway upgrade changed the default TLS cipher suites and old IoT clients can no longer connect; how do you detect the affected client population and roll a compatibility policy without weakening security for modern clients?](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02506_A_gateway_upgrade_changed_the_default_TLS_cipher_suites_and_.md>)
- [Deep] [Explain how you would implement mutual authentication and per-partner throttling for a B2B API where each partner has different SLAs and rate limits at the gateway.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02916_Explain_how_you_would_implement_mutual_authentication_and_pe.md>)
- [Deep] [Your gateway is the single external surface and Security wants a WAF inline, but the WAF adds latency and false-positives on legit API traffic; adjudicate WAF-at-gateway versus WAF-as-a-separate-hop versus schema-validation-as-WAF, and how you tune to avoid blocking real customers.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02745_Your_gateway_is_the_single_external_surface_and_Security_wan.md>)
- [Expert] [Your gateway must support 30k tenant-specific vanity domains with automated cert issuance and per-tenant policy; design the control-plane architecture for cert lifecycle, SNI routing scale, and the blast radius if the cert automation fails for a large cohort.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02591_Your_gateway_must_support_30k_tenant-specific_vanity_domains.md>)
- [Expert] [Your gateway must migrate from IP-based trust to workload-identity (SPIFFE/mTLS) for all upstream calls across hundreds of services; sequence this multi-year migration so there's never a window where valid workloads are rejected or unauthenticated ones slip through.](<../../../../PRACTICE/07_Distributed_Systems_and_Design/APIs/API_Gateway/Q-02651_Your_gateway_must_migrate_from_IP-based_trust_to_workload-id.md>)

## Related concepts

- [Org-wide API Standards](<../../API_Strategy/Org-wide_API_Standards.md>) *(API Strategy)*
- [API Rate Limiting](<../../REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*
- [Authorization Design](<../../REST_APIs/Authorization_Design.md>) *(REST APIs)*

