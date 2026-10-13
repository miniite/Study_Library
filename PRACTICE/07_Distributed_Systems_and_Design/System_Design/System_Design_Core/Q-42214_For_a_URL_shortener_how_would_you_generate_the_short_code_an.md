---
type: "practice"
difficulty: "Intermediate"
---

# For a URL shortener how would you generate the short code and avoid collisions?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ URL Shortener](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/URL_Shortener.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

This is the deep-dive follow-up to the URL-shortener design: "how do you generate the short code and avoid collisions?" The interviewer is zooming in on the single most subtle part of the system. At intermediate level they want to see that you:

Know the candidate techniques — hashing, random generation, and counter/ID-based encoding — and can compare them.
Understand why collisions happen and how each technique either prevents them by construction or detects-and-retries.
Can reason about distributed uniqueness — the code must be unique even though many app servers generate codes concurrently.
Handle the extras: custom aliases, non-guessability, and code length.

Clarifying questions: Must codes be unpredictable (security) or just unique? Do we support custom aliases? Rough scale (this sets the key-space)? I'll assume ~6B URLs over 5 years, custom aliases supported, and "not trivially enumerable" desired but not cryptographically secret.

The alphabet and length first

We encode codes in Base62 (0-9, a-z, A-Z). Length n gives 62^n codes:

62^6 ≈ 56.8 billion
62^7 ≈ 3.5 trillion

For 6B URLs, 7 characters leaves ~500× headroom, so collisions stay rare regardless of technique. (Base64 would need URL-safe chars -/_; Base62 avoids ambiguity, so it's the standard pick.)

Technique 1 — Hashing the long URL

Take MD5(longUrl) (or SHA-256), then take the first 7 Base62 characters. Simple and stateless.

Problems: MD5 is 128 bits; truncating to ~42 bits (7×log2(62)) throws away most of it, so by the birthday paradox collisions appear far sooner than the raw key-space suggests. With k codes and space N=62^7, expected collisions ≈ k²/2N; at 6B codes that's a lot of expected collisions. Also, hashing the same long URL twice yields the same code — good for dedup, bad if two users want independent links, and it leaks that the URL is identical. So hashing requires a collision-check-and-retry: compute hash → try to insert → on unique-constraint violation, append a salt/counter and rehash. Works, but every write may cost extra DB round-trips as the table fills.

Technique 2 — Random generation

Generate 7 random Base62 chars (from a CSPRNG). This gives non-guessable codes for free — a big security win over sequential ones. But randomness collides (birthday paradox again), so you must do an atomic "insert-if-not-exists" and retry on conflict. As the table fills toward the key-space, retry rate climbs — mitigate by keeping the space far larger than the population (7 chars for 6B is fine: fill factor ~0.0000017, so retries are essentially never). This is a clean, popular choice when unpredictability matters and scale is comfortably below the key-space.

Technique 3 — Counter / ID encoding (my default)

Maintain a globally unique, monotonically increasing integer and Base62-encode it. Because each integer is used once, codes are unique by construction — zero collisions, zero DB check. The only real problem is generating that unique integer across many servers without a bottleneck. Standard solutions:

Range/ticket allocation: a central service (or a DB sequence, or Zookeeper) hands each app server a block of IDs (e.g., 1,000–2,000 at a time). Servers hand out from their block locally and only call the allocator when the block runs out. This removes per-request coordination — one network hop per 1,000 codes.
Snowflake-style IDs: compose the integer from timestamp | machineId | sequence. Unique across machines with no coordination at all; downside is longer codes because the integer is large.
Redis INCR: atomic global counter; simple but a single hot key (mitigate with multiple counters / sharded ranges).

Guessability fix: a plain counter yields sequential codes (...aXk9Q1, ...aXk9Q2) so an attacker can enumerate. Mitigate by encoding with a shuffled/secret Base62 alphabet or applying a reversible bijection (e.g., a Feistel/XOR+multiply permutation, or Hashids) to the integer before encoding. That keeps uniqueness (bijection preserves it) while scrambling the visible order.

Custom aliases

For vanity URLs (sho.rt/my-brand), the user supplies the code. Just try to insert with a unique constraint; on conflict, reject with 409 Conflict and ask for another. Validate charset/length and blocklist reserved/offensive words. Aliases and generated codes share the same table and unique index, so a generated code will never clash with an existing alias (the insert simply fails and we regenerate — but with the counter approach, generated codes come from a numeric range you can keep disjoint from typical vanity strings).

Handling a collision when it does occur

Whatever technique, the DB unique constraint on short_code is the last line of defence. The generate step is a suggestion; the insert is the source of truth:

Generate candidate code.
INSERT ... ON CONFLICT DO NOTHING (or catch the unique-violation).
If it conflicted, regenerate (new random / next counter value / salted rehash) and retry, bounded to a few attempts.

With the counter approach step 3 essentially never fires; with hashing/random it fires rarely and cheaply.

Summary — what a strong answer emphasizes

State the three techniques and pick decisively: counter → Base62 with a bijective scramble is the go-to because it is collision-free by construction and scales via range allocation (or Snowflake) so no per-request coordination is needed; add the shuffle/Hashids step to kill guessability. If unpredictability is paramount and scale is well under the key-space, random + insert-if-not-exists is a fine, simpler alternative. Always keep a unique index as the ultimate guarantee and a bounded retry loop. Size the code at 7 Base62 chars (3.5T space) so the fill factor stays tiny. Custom aliases are just insert-with-unique-constraint plus validation.

#### Common mistakes
relying on hash truncation without a collision check (birthday paradox bites), using a naked counter (enumerable), a single global INCR as a bottleneck, or forgetting that the DB unique constraint — not the generator — is what actually guarantees uniqueness. Follow-up probes: how range allocation survives a server crash (you leak a block of IDs — acceptable), how to make codes unpredictable without losing uniqueness, and what code length you'd pick and why.

**Comparison:**

```
Technique	Collisions	Guessable?	Distributed uniqueness	Notes
Hash(url)→first 7	Yes (birthday paradox on truncation)	Somewhat	Stateless	Needs check+salt-retry; dedups identical URLs
Random Base62	Yes, but rare if space≫population	No (CSPRNG)	Stateless + insert-if-absent	Simple; retry rate rises as table fills
Counter→Base62	None (unique by construction)	Yes (sequential) → fix with scramble	Range allocation / Snowflake	My default; add bijection for unpredictability
Custom alias	User-driven	N/A	Unique index	409 on conflict + validation/blocklist
```

**Code:**

```python
import string
BASE62 = string.digits + string.ascii_lowercase + string.ascii_uppercase  # 62 chars

def encode_base62(n: int) -> str:
    if n == 0:
        return BASE62[0]
    out = []
    while n:
        n, r = divmod(n, 62)
        out.append(BASE62[r])
    return "".join(reversed(out))

# --- Technique 3: counter + range allocation (collision-free) ---
class RangeAllocator:
    """Each app server pulls a block of IDs, hands them out locally."""
    def __init__(self, allocator, block=1000):
        self.allocator, self.block = allocator, block
        self.next_id, self.end = 0, 0
    def next(self) -> int:
        if self.next_id >= self.end:
            self.next_id = self.allocator.reserve_block(self.block)  # 1 hop / 1000
            self.end = self.next_id + self.block
        nid = self.next_id
        self.next_id += 1
        return nid

def make_code(alloc: RangeAllocator) -> str:
    seq = alloc.next()
    scrambled = feistel_permute(seq)      # reversible bijection -> non-guessable
    return encode_base62(scrambled).rjust(7, "0")

# --- Technique 2: random + insert-if-not-exists (retry on conflict) ---
import secrets
def random_code(n=7) -> str:
    return "".join(secrets.choice(BASE62) for _ in range(n))

def create_random(db, long_url, max_tries=5):
    for _ in range(max_tries):
        code = random_code()
        if db.insert_if_absent(code, long_url):   # atomic, returns False on conflict
            return code
    raise RuntimeError("code space too full; increase length")
```

**Example input / output:**

```text
Counter technique (collision-free):
  seq=125,000,000  -> scramble -> encode_base62 -> "8m0Kf" -> padded "008m0Kf"
  seq=125,000,001  -> scramble -> "Qz3aP1"  (order scrambled, not sequential)

Random technique with a conflict:
  try 1: "b7Kd2Rp" -> INSERT -> ON CONFLICT (already exists) -> 0 rows -> retry
  try 2: "9pQ4wLm" -> INSERT -> success -> return "9pQ4wLm"

Custom alias:
  POST /shorten {alias:"my-brand"} -> INSERT 'my-brand'
     -> conflict -> 409 "alias taken, pick another"
```

**Data:**

```sql
-- Unique constraint is the ultimate collision guard (all techniques rely on it)
CREATE TABLE urls (
    short_code  VARCHAR(10) PRIMARY KEY,
    long_url    TEXT NOT NULL,
    created_at  TIMESTAMP DEFAULT now()
);
-- Collision-safe insert; regenerate on conflict
INSERT INTO urls (short_code, long_url)
VALUES ('aXk9Q2', 'https://example.com/...')
ON CONFLICT (short_code) DO NOTHING;   -- 0 rows affected => collision => retry
```

**Flow:**

1. Counter
2. Random
3. Hash
4. No
5. Yes
6. Request to create code
7. Technique
8. Get unique int via range alloc
9. Bijective scramble
10. Base62 encode -> 7 chars
11. CSPRNG 7 Base62 chars
12. MD5 url -> first 7 chars
13. INSERT with UNIQUEconstraint
14. Conflict?
15. Return code
16. Regenerate / next int / salt

