---
type: "practice"
difficulty: "Deep"
---

# Your gateway's JWKS cache refreshes on a timer, but the IdP rotated keys off-schedule and valid tokens started failing; how do you design a refresh-on-miss with backoff so an unknown kid triggers a controlled JWKS re-fetch?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Caching and Token Validation](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Caching_and_Token_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an API Gateway to handle JWT validation, it's crucial to ensure that the JWK (JSON Web Key) cache is refreshed in a robust and controlled manner. However, if the IdP rotates keys off-schedule, cached JWKs can become invalid, leading to token verification failures. The goal here is to implement a refresh-on-miss mechanism with backoff to handle unknown or missing JWKs while ensuring that the gateway remains resilient and responsive.

#### Triage
Understanding the Problem

The question requires us to design an efficient cache management strategy for JWKS (JSON Web Key Set) where:

The cache should be refreshed on a timer.
If the cache misses (i.e., the key is unknown), it should trigger a controlled re-fetch of JWKs.
There must be backoff logic in place to prevent overwhelming the IdP with too many requests.

#### Assumptions
and Context
The gateway uses a timer-based approach for refreshing the JWKS cache.
On-miss handling needs to be robust, ensuring that unknown or missing keys are handled gracefully without causing service disruptions.
Backoff should help manage the rate of re-fetches to avoid overloading the IdP.

#### First Principles Solution

Timer-Based Cache Refresh:

Implement a background task that runs periodically (e.g., every 30 minutes) to refresh the JWKS cache from the IdP.

Refresh-On-Miss Handling:

When token validation encounters an unknown or missing key, instead of failing immediately, it should trigger a delayed re-fetch.
Use a backoff strategy to ensure that subsequent fetches are spaced out (e.g., 10 seconds, then 30 seconds, then 60 seconds).

Controlled Re-Fetch Logic:

Implement a mechanism where the gateway can detect when keys have been rotated off-schedule and start controlled re-fetches.
Use retries with exponential backoff to handle transient failures.
Detailed Design

Timer Setup:

PYTHON
import time
from threading import Timer

class JWKSRefreshTask:
    def __init__(self, gateway):
        self.gateway = gateway
        self.timer = None

    def start(self):
        if self.timer is not None and self.timer.is_alive():
            self.timer.cancel()
        self.timer = Timer(1800, self.fetch_jwks)  # 30 minutes
        self.timer.start()

    def fetch_jwks(self):
        try:
            self.gateway.fetch_and_cache_jwks()
            self.start()  # Reset timer for next refresh
        except Exception as e:
            print(f"Failed to fetch JWKS: {e}")

Fetch and Cache JWKs:

PYTHON
import requests

class APIGateway:
    def __init__(self):
        self.jwks_cache = {}

    def fetch_and_cache_jwks(self):
        jwks_url = "https://example.com/jwks"  # IdP's JWKS URL
        response = requests.get(jwks_url)
        if response.status_code == 200:
            self.jwks_cache = response.json()

Refresh-On-Miss with Backoff:

PYTHON
class JWTValidator:
    def __init__(self, gateway):
        self.gateway = gateway

    def validate_token(self, token):
        try:
            jwk_id = get_jwk_id_from_token(token)  # Some function to extract JWK ID from the token
            if jwk_id in self.gateway.jwks_cache:
                verify_token_with_jwk(jwk_id, token)
            else:
                # Trigger a re-fetch with backoff
                self.handle_re_fetch(jwk_id)
        except Exception as e:
            print(f"Token validation failed: {e}")

    def handle_re_fetch(self, jwk_id):
        if jwk_id not in self.gateway.jwks_cache:
            backoff = 10  # Initial delay
            for attempt in range(5):  # Max 5 attempts with backoff
                try:
                    self.gateway.fetch_and_cache_jwks()
                    if jwk_id in self.gateway.jwks_cache:
                        verify_token_with_jwk(jwk_id, token)
                        break
                except Exception as e:
                    print(f"Attempt {attempt + 1} failed: {e}")
                time.sleep(backoff)
                backoff *= 2  # Double the delay for each retry

Controlled Re-Fetch Logic:

PYTHON
def handle_key_rotation(self, jwks_url):
    if is_off_schedule(jwks_url):  # Some function to check if keys are rotated off-schedule
        self.gateway.fetch_and_cache_jwks()

#### Closing Summary
and Common Mistakes

This solution ensures that the JWKS cache is refreshed in a controlled manner, with fallback mechanisms for handling unknown or missing keys. The backoff strategy helps manage the load on the IdP while ensuring that key rotation issues are promptly addressed.

#### Common Mistakes

Forgetting to handle exceptions properly, leading to service disruptions.
Not implementing retry logic, which could lead to unnecessary failures and retries.
Failing to consider edge cases where the IdP might be temporarily unavailable or overly responsive.
Common Follow-Ups
How do you ensure that the cache is invalidated when a key rotation occurs?
What measures are in place to prevent Denial of Service (DoS) attacks on the IdP?
Can you describe how the gateway handles edge cases where network connectivity issues occur?

