---
type: "practice"
difficulty: "Deep"
---

# How do you prevent OTP brute-force and abuse?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Notification System](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Notification_System.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: "OTP brute-force and abuse" bundles two distinct threat classes that a strong candidate must separate. (1) Verification-side attacks — an attacker who has (or has phished) a phone number/email tries to guess the OTP by submitting many attempts. (2) Request-side abuse — an attacker spams the send-OTP endpoint to burn your SMS budget (an "SMS pumping"/toll-fraud attack, very real for fintechs like Razorpay/PhonePe), enumerate accounts, or grief users by flooding their phones. The interviewer (fintech/payments/cloud tier) is probing whether you can reason about entropy math, rate limiting at multiple layers, atomicity under concurrency, and cost/abuse economics — not just "add a rate limiter."

Clarifying questions I'd state up front:

What's the OTP length and channel (SMS/email/authenticator)? 6-digit numeric SMS is the default assumption.
Is this login, signup, txn-authorization, or password reset? (Txn OTPs need the tightest controls.)
Expected legit volume and geography (drives cost of SMS pumping)?
Do we already have Redis/a shared store? (Needed for cross-node counters.)

Plan: first do the entropy/attempt math to size the limits, then design the two independent control planes (send-side and verify-side), then handle the distributed-systems correctness (atomic decrement, single-use, hashing), then layer on abuse-economics defenses (device/IP reputation, CAPTCHA, cost caps), then failure modes.

From first principles: the entropy math

A 6-digit numeric OTP has 10^6 = 1,000,000 possible values. A single blind guess succeeds with probability 1/1,000,000. If we allow N verification attempts before locking, the cumulative success probability is ≈ N/10^6. So:

Allow 5 attempts → ~1-in-200,000 chance per OTP. Acceptable.
Allow 1,000 attempts → 1-in-1,000. Not acceptable.

This is why the single most important control is a hard cap on verify attempts per OTP, combined with a short TTL (30–120s). TTL matters because it bounds how long the guessing window stays open and how many OTPs an attacker can farm. Together: few attempts × short life = brute force is economically dead.

For higher-value flows (payment authorization) use a longer OTP or alphanumeric to raise entropy, but the attempt cap does the heavy lifting.

Control plane 1 — Verify-side (stop guessing)
Per-OTP attempt counter with atomic decrement. Store, keyed by the identifier + OTP session id: {otp_hash, attempts_left, expires_at}. On each verify, atomically decrement and reject at 0. Do this in Redis with INCR/DECR or a Lua script so concurrent guesses from many nodes can't race past the cap (see code).
Invalidate on success AND on exhaustion. A correct OTP is single-use — delete it immediately so a replay fails. When attempts hit 0, delete the OTP so even the real code no longer works; the user must request a new one.

#### Cons
tant-time compare + hashed storage. Never store the raw OTP; store HMAC(otp, server_secret) or a salted hash. Compare in constant time to avoid timing side-channels. This also protects OTPs if the store leaks.
Progressive lockout on the account/identifier, not just the OTP: e.g. after 3 failed OTP sessions, require CAPTCHA; after more, exponential backoff / temporary lock. This defeats "request new OTP, guess 5, repeat" farming.
Control plane 2 — Send-side (stop spamming / SMS pumping)
Rate limit send-OTP by multiple keys, ANDed: per phone/email, per IP, per device fingerprint, and a global ceiling. Keying only by IP is useless behind CGNAT/botnets; keying only by phone lets an attacker rotate phones to pump SMS. Typical: max 1 OTP / 60s and 5 / hour per number.
Resend cooldown + reuse the live OTP. If a valid OTP already exists and isn't near expiry, resend the same code instead of minting a new one — this alone kills most spam and halves SMS spend.
CAPTCHA / proof-of-work gate on the send endpoint once a soft threshold is crossed, so bots can't cheaply loop.
Cost/abuse caps for SMS pumping: block or throttle high-risk destination country codes and premium-rate ranges (attackers route OTPs to numbers they own to collect carrier revenue-share), cap spend per country/hour with alerting, and prefer a provider with pumping protection. This is the attack that actually costs fintechs money.
Silent handling for enumeration: send-OTP and verify responses must not reveal whether an account exists ("If an account exists, a code was sent") to prevent account enumeration.
Distributed / concurrency correctness
The counter store must be shared and atomic across all API nodes (Redis, DynamoDB conditional writes) — an in-memory per-process counter is trivially bypassed by hitting different nodes.
Use atomic operations (Lua/WATCH-MULTI or a single DECR) so 100 parallel guesses can't each read "5 left." (See code block.)
Set TTLs on every key so limits self-heal and Redis doesn't leak keys.
Defense in depth / abuse economics

Layer WAF + edge rate limiting (block obvious floods before your app), device-fingerprint reputation, and anomaly detection (spikes in send volume, low verify-success ratio → bot). The goal is to make an attack cost more than it yields — brute force needs many attempts (blocked by the cap), and pumping needs many sends (blocked by cost caps + CAPTCHA).

Failure modes & follow-ups
Redis down: fail closed on verify (don't grant access if you can't count attempts) but allow a degraded send with tight defaults — decide per business risk. A payments flow should fail closed everywhere.
Legit user locked out: always provide an alternate channel (voice call, email) and clear "try again in X" messaging; make lockout time-boxed, not permanent.
Clock skew across nodes: rely on Redis TTL (server-authoritative), not client time.
What a strong answer emphasizes

Two independent control planes (send vs verify), the attempt-cap + short-TTL math as the core brute-force defense, atomic cross-node counters, single-use + hashed OTPs, and — the differentiator for a fintech interview — SMS-pumping cost defense and account-enumeration silence. Common mistakes: only rate-limiting by IP, using an in-memory counter, storing raw OTPs, allowing unlimited resends, and forgetting to invalidate the OTP on success/exhaustion.

**Comparison:**

```
Threat	Vector	Primary defense	Secondary
Brute-force guess	Many verify attempts	Hard attempt cap (5) + short TTL (120s)	Progressive account lockout
Replay	Reuse consumed OTP	Single-use: delete on success	Bind OTP to session/nonce
SMS pumping / toll fraud	Spam send to attacker-owned numbers	Country-code + spend caps, provider pumping protection	CAPTCHA gate
Spam / grief user	Flood send to a victim	Per-number cooldown (60s) + hourly cap + resend-reuse	Device fingerprint rep
Account enumeration	Response differs if account exists	Uniform "if account exists…" response	Same latency for both paths
Distributed bypass	Hit many API nodes	Shared atomic store (Redis Lua)	WAF/edge rate limit
Control	Value (typical)	Rationale
OTP length	6 digits (10^6)	Balance UX vs entropy; cap does the real work
TTL	30–120 s	Short guessing window
Verify attempts	5 per OTP	~1-in-200k success
Sends per number	1/60s, 5/hour	Anti-spam + cost
Storage	HMAC/hashed, not raw	Survive store leak
```

**Code:**

```

-- verify_otp.lua (Redis EVAL: atomic, race-free attempt counting) -- KEYS[1] = otp session key, e.g. "otp:login:+9198xxxx" -- ARGV[1] = HMAC(submitted_otp) ARGV[2] = max_attempts (unused here, TTL/attempts pre-seeded) local raw = redis.call('HGETALL', KEYS[1]) if #raw == 0 then return 'EXPIRED' end -- no live OTP / already used

local h = redis.call('HGET', KEYS[1], 'hash') local left = tonumber(redis.call('HGET', KEYS[1], 'left'))

if left <= 0 then redis.call('DEL', KEYS[1]) -- exhausted: kill even the real code return 'LOCKED' end

if h == ARGV[1] then redis.call('DEL', KEYS[1]) -- correct: single-use, consume it return 'OK' end

left = redis.call('HINCRBY', KEYS[1], 'left', -1) -- wrong: atomic decrement if left <= 0 then redis.call('DEL', KEYS[1]); return 'LOCKED' end return 'WRONG'

CODE
Copy

```javascript
// Node/Express-ish send + verify wiring (Redis client `r`)
const crypto = require('crypto');
const hmac = (otp) => crypto.createHmac('sha256', process.env.OTP_SECRET)
                            .update(otp).digest('hex');

async function sendOtp(id, ip) {
  // send-side rate limits, ANDed: per-id and per-ip, plus resend cooldown
  const idKey = `rl:send:id:${id}`, ipKey = `rl:send:ip:${ip}`;
  const sends = await r.incr(idKey); if (sends === 1) await r.expire(idKey, 3600);
  if (sends > 5) throw new Error('RATE_LIMITED');          // 5 / hour / id
  const ipc = await r.incr(ipKey); if (ipc === 1) await r.expire(ipKey, 3600);
  if (ipc > 30) throw new Error('RATE_LIMITED');           // per-IP ceiling

  const sess = `otp:login:${id}`;
  if (await r.exists(sess) && await r.ttl(sess) > 90) return; // reuse live OTP (anti-spam)

  const otp = String(crypto.randomInt(0, 1_000_000)).padStart(6, '0');
  await r.hset(sess, { hash: hmac(otp), left: 5 });         // 5 verify attempts
  await r.expire(sess, 120);                                // 120s TTL
  await sms.send(id, `Your code is ${otp}`);                // + country-code cost caps upstream
}

async function verifyOtp(id, otp) {
  const res = await r.eval(VERIFY_LUA, 1, `otp:login:${id}`, hmac(otp), 5);
  return res; // 'OK' | 'WRONG' | 'LOCKED' | 'EXPIRED'
}
```

**Example input / output:**

```text
Scenario A — brute force guessing (attacker knows the number, not the code)
  POST /verify {id:+91..., otp:000000} -> WRONG  (left 5->4)
  POST /verify {id:+91..., otp:000001} -> WRONG  (left 4->3)
  ... 3 more wrong -> left hits 0 -> LOCKED, OTP deleted
  Result: <=5 guesses per code, ~1-in-200k success; new code needs a send (rate-limited).

Scenario B — 100 concurrent guesses on different API nodes
  All hit the same Redis EVAL; HINCRBY is atomic -> exactly 5 decrements honored,
  the 6th+ see left<=0 -> LOCKED. No race lets attempt 6..100 through.

Scenario C — SMS pumping / spam
  6th send in an hour for one number -> RATE_LIMITED (no SMS sent).
  Resend within 90s of TTL -> same code reused, zero extra SMS cost.
  Sends to a flagged premium-rate country code -> blocked by cost cap.

Scenario D — replay after success
  Correct OTP -> OK, key DELeted. Re-submitting same OTP -> EXPIRED.
```

**Data:**

```json
// Redis hash stored per OTP session (key: otp:login:<identifier>, TTL 120s)
{
  "hash": "9f2b...HMAC-SHA256 of the 6-digit code, never the raw code",
  "left": 5
}

// Rate-limit counters (all TTL'd so limits self-heal)
// rl:send:id:+919812345678  -> 3   (EX 3600)  max 5/hr per number
// rl:send:ip:203.0.113.7    -> 12  (EX 3600)  per-IP ceiling
// lock:acct:+919812345678   -> 1   (EX 900)   progressive account lockout
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. no
6. yes
7. no
8. yes
9. no
10. yes
11. yes
12. no
13. POST /send-otp
14. Per-id + per-IP + globalrate limits OK?
15. 429 RATE_LIMITEDuniform response
16. Live OTP exists& TTL>90s?
17. Resend SAME codeno new SMS
18. Country/cost cap OK?CAPTCHA if suspicious
19. Gen OTP -> store HMAC,left=5, TTL=120s -> send SMS
20. POST /verify-otp
21. Redis EVAL atomic
22. key exists?
23. EXPIRED
24. left>0?
25. LOCKED + DEL
26. HMAC match?
27. OK + DEL single-use
28. DECR left; if 0 -> LOCKED+DELelse WRONG

