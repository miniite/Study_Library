---
id: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design"
level: 5
type: "concept"
topic: "07_Distributed_Systems_and_Design/System_Design/High_Level_Design"
tools: ["Redis"]
---

# Distributed Rate Limiter Design

**L5 Expert** · [↑ High Level Design](<_index.md>)

<!-- explainer:start -->
> _Lesson pending._
<!-- explainer:end -->

## Implementations (tools)

- [Redis](<../../../TOOLS/Redis/_index.md>)

## Practice questions

- [Intermediate] [You need per-user rate limiting enforced partly on the client (to give instant UI feedback) and authoritatively on the server — how do you split and reconcile the two?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29605_You_need_per-user_rate_limiting_enforced_partly_on_the_clien.md>)
- [Intermediate] [Design an OTP-generation-and-verification service used across products — how do you rate-limit per number resend prevent enumeration and store OTPs securely with short TTL at scale?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29752_Design_an_OTP-generation-and-verification_service_used_acros.md>)
- [Deep] [Design a distributed rate limiter used by an API gateway across 50 nodes — compare token bucket, leaky bucket, and sliding-window-log, and where you store the state.](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29603_Design_a_distributed_rate_limiter_used_by_an_API_gateway_acr.md>)
- [Deep] [Design an email service (like sending transactional + marketing email at scale) — how do you handle sender reputation, bounce/complaint feedback, and per-domain throttling?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29674_Design_an_email_service_like_sending_transactional_marketing.md>)
- [Deep] [Your Redis-backed rate limiter is the bottleneck at peak — its INCR calls are adding tail latency — how do you shard the limiter keys and batch increments while keeping the limit approximately correct?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29695_Your_Redis-backed_rate_limiter_is_the_bottleneck_at_peak_its.md>)
- [Deep] [A notification campaign to 40M users must respect a per-provider send rate and complete within a 2-hour window — how do you design the pacing/throttle scheduler so you neither burst-limit the provider nor miss the window?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29701_A_notification_campaign_to_40M_users_must_respect_a_per-prov.md>)
- [Deep] [Design a token/session revocation system for JWT-based auth where a logout or a compromised-token event must invalidate access globally within seconds — how do you avoid a per-request DB check?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29754_Design_a_token_session_revocation_system_for_JWT-based_auth_.md>)
- [Expert] [Your distributed rate limiter uses a central Redis — how do you keep it accurate under network partitions and avoid it becoming a single point of failure or a hotspot?](<../../../PRACTICE/07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Q-29604_Your_distributed_rate_limiter_uses_a_central_Redis_how_do_yo.md>)

## Related concepts

- [API Rate Limiting](<../../APIs/REST_APIs/API_Rate_Limiting.md>) *(REST APIs)*
- [Rate Limiter Storage](<../System_Design_Core/Rate_Limiter_Storage/_index.md>) *(System Design Core)*
- [Rate Limiting at the Gateway](<../../APIs/API_Gateway/Rate_Limiting_at_the_Gateway.md>) *(API Gateway)*

