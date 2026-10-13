---
type: "practice"
difficulty: "Deep"
---

# Your pipeline's cache-key logic restored a stale node_modules and green builds shipped broken code; how would you design cache keys to be correct-by-construction and detect poisoning before it ships?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ CI Caching](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that our CI/CD pipeline is both correct-by-construction and capable of detecting poisoned artifacts, we need a robust caching strategy that considers all relevant dependencies and metadata. The key idea is to design cache keys in such a way that they are deterministic, yet flexible enough to capture any changes. This ensures that the cached artifact is only used when it corresponds exactly to the inputs, thus preventing stale or poisoned node_modules from causing issues.

#### Triaging the Question

The core of this question revolves around designing cache keys for CI/CD pipelines to avoid using stale artifacts and to ensure that any tampering (poisoning) is detected. The interviewer is probing our understanding of:

Deterministic Cache Keys: Ensuring cache keys are based on a consistent set of inputs.
Dependency Management: Handling dynamic dependencies like node_modules effectively.
Poison Detection Mechanisms: Implementing mechanisms to detect and prevent the use of poisoned artifacts.

#### Assumptions
The pipeline is using Docker containers for consistency across environments.
Node.js is the primary runtime, with npm as the package manager.
Cache keys are computed based on a combination of input files and their hashes.

#### First Principles Solution

Compute Cache Keys Based on Input Files:

Include all relevant input files in the cache key computation process. For node_modules, consider using directory snapshots or checksums of package.json and npm-shrinkwrap.json (if applicable).

Dynamic Dependency Handling:

Ensure that any changes to dependencies are reflected in the cache keys. This includes updates to package.json, npm-shrinkwrap.json, and other relevant configuration files.

Environment-specific Keys:

Include environment variables or build tags as part of the cache key computation. This ensures that artifacts built on different environments (e.g., dev vs prod) are not accidentally reused.

Poison Detection Mechanisms:

Implement a signature-based mechanism to detect tampering. For example, generate a cryptographic hash of the cached artifact and store it alongside the artifact.
During build time, recompute this hash and verify it against the stored value.

Version Control Integration:

Use version control system (VCS) metadata as part of the cache key to ensure that only versions tagged with specific revisions are used.

Regular Audits and Logging:

Regularly review build logs and cache keys to detect any anomalies or tampering attempts.

#### Example
Cache Key Computation
PYTHON
def compute_cache_key():
    # Include package.json and npm-shrinkwrap.json checksums
    package_json_checksum = hashlib.sha256(open('package.json', 'rb').read()).hexdigest()
    shrinkwrap_json_checksum = hashlib.sha256(open('npm-shrinkwrap.json', 'rb').read()).hexdigest()

    # Include environment variables or build tags
    environment_tag = os.environ.get('BUILD_ENV')

    # Combine all elements into a single cache key
    return f"{package_json_checksum}_{shrinkwrap_json_checksum}_{environment_tag}"
Cache Key Verification During Build
PYTHON
def verify_cache_key():
    stored_hash = get_stored_hash_from_vcs()  # Retrieve from VCS or cache storage
    computed_hash = compute_cache_key()

    if stored_hash != computed_hash:
        raise Exception("Cache key mismatch detected, potential tampering!")

#### Common Mistakes
and Follow-ups
Stale Cache Usage: Not computing the cache keys based on all relevant inputs can lead to stale artifacts being used.
No Poison Detection: Failing to implement mechanisms like cryptographic hashes for detecting tampered artifacts.
Inconsistent Environment Handling: Not including environment-specific details in cache keys can cause issues across different build environments.

#### Closing Summary

Designing correct-by-construction cache keys and implementing robust poison detection mechanisms are crucial steps in ensuring the reliability of CI/CD pipelines. By carefully considering all relevant inputs, integrating VCS metadata, and verifying the integrity of artifacts, we can significantly reduce the risk of shipping broken or poisoned code.

**Code:**

```python
import hashlib

def compute_cache_key():
    package_json_checksum = hashlib.sha256(open('package.json', 'rb').read()).hexdigest()
    shrinkwrap_json_checksum = hashlib.sha256(open('npm-shrinkwrap.json', 'rb').read()).hexdigest()
    environment_tag = os.environ.get('BUILD_ENV')

    return f"{package_json_checksum}_{shrinkwrap_json_checksum}_{environment_tag}"

def verify_cache_key():
    stored_hash = get_stored_hash_from_vcs()  # Retrieve from VCS or cache storage
    computed_hash = compute_cache_key()

    if stored_hash != computed_hash:
        raise Exception("Cache key mismatch detected, potential tampering!")
```

**Flow:**

1. Matched
2. Mismatch Detected
3. Compute Cache Key
4. Verify Hash
5. Proceed with Build
6. Report Tampering Attempt andAbort Build

