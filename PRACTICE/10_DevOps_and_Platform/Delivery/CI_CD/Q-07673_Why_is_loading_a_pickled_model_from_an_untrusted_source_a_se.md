---
type: "practice"
difficulty: "Deep"
---

# Why is loading a pickled model from an untrusted source a security risk, and how would you gate against it in the pipeline?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Secrets in Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Loading a pickled model from an untrusted source can pose significant security risks because pickle files can contain arbitrary code that is executed when loaded. This means that if an attacker manages to inject malicious data into the pickle file, they could exploit vulnerabilities in the model's deserialization process to execute harmful code or access sensitive information.

To gate against this risk in a CI/CD pipeline, we need to take several steps from first principles:

Source Control and Versioning: Ensure that only trusted source repositories are used for model storage.
Model Signing and Verification: Implement a signing mechanism where models are digitally signed before being deployed or loaded. The verification process can be integrated into the CI/CD pipeline.
Input Validation: Validate the input data to ensure it adheres to expected formats and does not contain malicious content.
Environment Hardening: Ensure that environments running the model are isolated and hardened against attacks.

Let's break this down step by step:

#### Triage

The question asks about the security risks of loading pickled models from untrusted sources and how to mitigate these risks in a CI/CD pipeline. The core idea is to prevent unauthorized code execution and protect sensitive information.

#### Assumptions
We are using Python's pickle module for model serialization.
The environment where the model is loaded is a production system or another critical system that must remain secure.
There is an existing CI/CD pipeline in place.
Step-by-Step Solution

Source Control and Versioning

Ensure that models are stored in version-controlled repositories (e.g., Git).
Implement a robust branching strategy to control who can commit changes to the model storage repository.

Model Signing and Verification

Use digital signatures for model files.
Integrate a verification step into the CI/CD pipeline that checks the signature before loading the model.

Input Validation

Define clear input validation rules at both training and deployment stages.
Implement a validation function that ensures all inputs are sanitized and do not contain malicious payloads.

Environment Hardening

Ensure isolation of the environment where models are loaded.
Use containerization to create isolated environments for model loading and execution.

#### Implementation
in CI/CD Pipeline

Source Control and Versioning

BASH
# Example Git commit message
git commit -m "Updated model with new version"

Model Signing and Verification

PYTHON
import hashlib
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.backends import default_backend

def sign_model(model_bytes, private_key):
    digest = hashes.Hash(hashes.SHA256(), backend=default_backend())
    digest.update(model_bytes)
    signature = private_key.sign(digest.finalize(), padding.PSS(mgf=padding.MGF1(hashes.SHA256()), salt_length=padding.PSS.MAX_LENGTH))
    return signature

def verify_model(model_bytes, public_key):
    digest = hashes.Hash(hashes.SHA256(), backend=default_backend())
    digest.update(model_bytes)
    try:
        public_key.verify(digest.finalize(), model_bytes, padding.PSS(mgf=padding.MGF1(hashes.SHA256()), salt_length=padding.PSS.MAX_LENGTH))
        return True
    except InvalidSignature:
        return False

Input Validation

PYTHON
import pickle

def validate_input(input_data):
    # Example validation rules (this can be more complex in real scenarios)
    if not isinstance(input_data, dict) or 'data' not in input_data:
        raise ValueError("Invalid input format")
    return True

def load_model(model_path):
    with open(model_path, "rb") as f:
        model_bytes = f.read()
        # Verify the signature
        public_key = serialization.load_pem_public_key(
            b"-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...\n-----END PUBLIC KEY-----",
            backend=default_backend()
        )
        if not verify_model(model_bytes, public_key):
            raise ValueError("Model signature verification failed")
        # Validate input
        validate_input(input_data)
    return pickle.loads(model_bytes)

Environment Hardening

BASH
docker run -v /models:/app/models my-model-container python3 app.py

#### Closing Summary
and Common Mistakes

To summarize, the key steps involve using version control for model storage, signing models to ensure their integrity, validating input data, and hardening the environment where models are loaded. These measures collectively reduce the risk of unauthorized code execution.

#### Common mistakes
include:

Overlooking the importance of digital signatures.
Not implementing thorough input validation.
Failing to isolate critical environments.
Neglecting regular security audits and updates in the CI/CD pipeline.

**Code:**

```python
import hashlib
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.backends import default_backend

def sign_model(model_bytes, private_key):
    digest = hashes.Hash(hashes.SHA256(), backend=default_backend())
    digest.update(model_bytes)
    signature = private_key.sign(digest.finalize(), padding.PSS(mgf=padding.MGF1(hashes.SHA256()), salt_length=padding.PSS.MAX_LENGTH))
    return signature

def verify_model(model_bytes, public_key):
    digest = hashes.Hash(hashes.SHA256(), backend=default_backend())
    digest.update(model_bytes)
    try:
        public_key.verify(digest.finalize(), model_bytes, padding.PSS(mgf=padding.MGF1(hashes.SHA256()), salt_length=padding.PSS.MAX_LENGTH))
        return True
    except InvalidSignature:
        return False

def validate_input(input_data):
    if not isinstance(input_data, dict) or 'data' not in input_data:
        raise ValueError("Invalid input format")
    return True

def load_model(model_path):
    with open(model_path, "rb") as f:
        model_bytes = f.read()
        public_key = serialization.load_pem_public_key(
            b"-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...\n-----END PUBLIC KEY-----",
            backend=default_backend()
        )
        if not verify_model(model_bytes, public_key):
            raise ValueError("Model signature verification failed")
        validate_input(input_data)
    return pickle.loads(model_bytes)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Valid model with correct signature and data validation | Model loaded successfully |
| Invalid model with incorrect signature or input validation failure | ValueError raised |

**Data:**

```json
{
  "public_key": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...\n-----END PUBLIC KEY-----"
}
```

**Flow:**

1. CI/CD Pipeline
2. Source Control and Versioning
3. Model Signing and Verification
4. Input Validation
5. Environment Hardening

