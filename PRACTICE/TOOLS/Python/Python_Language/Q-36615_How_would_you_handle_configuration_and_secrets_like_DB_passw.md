---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle configuration and secrets (like DB passwords) in a Python backend without hardcoding them?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Credential Handling](<../../../../TOOLS/Python/Python_Language/Credential_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: "How do you keep configuration (env-varying settings) and secrets (DB passwords, API keys, tokens) out of your code, and manage them safely across environments?" The interviewer — for an intermediate Backend/Fullstack SDE — is checking that I understand two things clearly: (1) the difference between config and secrets, and (2) the standard layered approach, from twelve-factor env vars up to a managed secrets store, plus the operational hygiene around it (rotation, no secrets in git, least privilege). At this level they want a practitioner answer: what I actually do, in what order, and the pitfalls I've hit.

Key distinction I'd state first:

Configuration = non-sensitive, environment-specific values (log level, feature flags, page size, service URLs, timeouts). Fine to keep in a versioned config file per environment.
Secrets = anything that grants access if leaked (DB password, API keys, JWT signing key, TLS private key). These must NEVER be in source control and ideally never on disk in plaintext.

Clarifying questions: Where does this run (single VM, containers/Kubernetes, serverless, a specific cloud)? Do we already have a cloud secrets manager (AWS Secrets Manager, Vault)? What's our rotation/compliance requirement? The best mechanism follows the platform, so I'll cover the general ladder and then the platform-specific top rung.

Reasoning from first principles — the layered approach

1. Twelve-Factor: config in the environment. The baseline rule (12-factor app, factor III) is store config in the environment, not in code. Code reads from os.environ. This lets the same build/image run in dev, staging, and prod with different values, and it keeps secrets out of the repo. In Python I centralize this rather than sprinkling os.getenv everywhere.

2. A typed settings object with validation. I load env vars once into a single settings class — using Pydantic Settings (pydantic-settings), or dynaconf. This gives type coercion ("5432" → int), defaults for non-secret config, required enforcement (the app fails fast at startup if DATABASE_URL is missing, instead of blowing up on the first query), and one import point for the whole app. Fail-fast on missing secrets is a real reliability win.

3. .env files for local dev only — and gitignored. Locally I use a .env file loaded via python-dotenv (Pydantic Settings loads it automatically). .env is in .gitignore; I never commit it. I commit a .env.example with keys and dummy values so teammates know what's required. Critically: .env files are a dev convenience only — they are NOT how prod secrets should live.

4. Production: a managed secrets store, injected at runtime. In real environments I don't ship secrets as plain files. The good options, roughly in increasing power:

Cloud secrets manager — AWS Secrets Manager / Parameter Store (SecureString), GCP Secret Manager, Azure Key Vault. App fetches at startup (or via an SDK cache) using its IAM role — no static credential to bootstrap, because the instance/pod identity authenticates. Supports automatic rotation and audit logging.
HashiCorp Vault — dynamic, short-lived DB credentials (Vault mints a per-app DB user that expires), fine-grained policies, full audit trail. Strongest option for regulated fintech.
Kubernetes: Secrets mounted as env vars or files, ideally sourced from a real store via External Secrets Operator or CSI Secrets Store driver so K8s isn't the source of truth (base64 in etcd is encoding, not encryption — I'd enable encryption-at-rest for etcd).
Serverless (Lambda): pull from Secrets Manager/Parameter Store at cold start, or use Lambda extensions to cache.

The pattern: the app authenticates to the store with a machine identity (IAM role / K8s service account), fetches the secret, holds it in memory. No plaintext secret on disk, none in the image, none in git.

5. Operational hygiene (what separates a mid-level answer from a junior one):

Never log secrets. Redact them in logs and error reports; scrub before sending to Sentry.
Rotation. Rotate keys/passwords on a schedule and immediately on suspected leak. Managed stores automate this; my code re-reads on rotation (short TTL cache) rather than caching forever.
Least privilege. Each service reads only its own secrets; scope IAM policies tightly. Prod credentials are separate from dev.
Secret scanning in CI. Run git-secrets / trufflehog / gitleaks (or GitHub push protection) to block accidental commits.
If a secret ever lands in git history, rotate it — deleting the commit is not enough, because it's already exposed.
Putting it together

My default: a Settings class (Pydantic) reads from os.environ; locally those come from a gitignored .env; in prod they're injected by the platform from a secrets manager via the service's IAM/service-account identity; config-vs-secret is separated; and I add rotation + secret-scanning + no-logging discipline around it.

Summary — what a strong answer emphasizes

Land these points: (1) separate config from secrets; (2) config in the environment, read through one validated settings object that fails fast; (3) .env is dev-only and gitignored, never prod; (4) prod secrets come from a managed store (Secrets Manager/Vault/Key Vault) fetched via machine identity, not hardcoded and not in the image; (5) surround it with rotation, least privilege, no-logging, and CI secret scanning.

#### Common mistakes
conflating config and secrets; thinking .env is a production solution; committing .env or forgetting .gitignore; base64 in K8s "counts" as encryption (it doesn't); and forgetting that a leaked secret must be rotated, not just removed from git.

**Comparison:**

```
Mechanism	Where it fits	Rotation	Secrets on disk?	Notes
.env + python-dotenv	Local dev only	Manual	Yes (gitignored)	Never for prod
OS env vars (12-factor)	Any runtime	Manual	No	Baseline; inject at deploy
AWS Secrets Manager / Parameter Store	AWS	Automatic	No (fetched to memory)	Auth via IAM role
GCP Secret Manager / Azure Key Vault	GCP / Azure	Automatic	No	Cloud-native equivalents
HashiCorp Vault	Any / regulated	Dynamic short-lived creds	No	Strongest; per-app DB users
K8s Secrets (+ External Secrets Operator)	Kubernetes	Via backing store	Optional (env/file)	Enable etcd encryption-at-rest
```

**Code:**

```
settings.py — one validated, fail-fast settings object for the whole app

from pydantic import Field, PostgresDsn from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings): # Loads from OS env; locally also reads a gitignored .env file. model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

CODE
Copy
# --- Config (non-sensitive, has defaults) ---
log_level: str = "INFO"
page_size: int = 20
request_timeout_s: float = 5.0

# --- Secrets (required: app refuses to start if absent) ---
database_url: PostgresDsn = Field(..., alias="DATABASE_URL")
jwt_signing_key: str = Field(..., alias="JWT_SIGNING_KEY")

settings = Settings() # raises at startup if a required secret is missing

CODE
Copy

```python
# In prod: fetch from AWS Secrets Manager at startup using the instance/pod IAM role.
# No static credential needed — the machine identity authenticates.
import json
import boto3

def load_db_password(secret_id: str) -> str:
    client = boto3.client("secretsmanager")  # auth via IAM role, no hardcoded key
    resp = client.get_secret_value(SecretId=secret_id)
    return json.loads(resp["SecretString"])["password"]
```

**Example input / output:**

```

Startup fail-fast example:

CODE
Copy
# Missing required secret in the environment:
$ unset DATABASE_URL && python -c "import settings"
pydantic_core._pydantic_core.ValidationError: 1 validation error for Settings
DATABASE_URL
  Field required [type=missing, ...]
# -> App refuses to boot instead of crashing on the first DB query. Good.
```

**Data:**

```
.gitignore — never commit real secrets

.env .env.* !.env.example *.pem *.key

CODE
Copy

```dotenv
# .env.example  (committed — placeholders only, so teammates know what's required)
LOG_LEVEL=INFO
PAGE_SIZE=20
DATABASE_URL=postgresql://user:CHANGE_ME@localhost:5432/appdb
JWT_SIGNING_KEY=replace-with-local-dev-key
```

**Flow:**

1. Prod
2. gitignored .env
3. fetch at startup, cache inmemory
4. never
5. never plaintext
6. Local dev
7. App
8. Pod / instance IAM identity
9. (Secrets Manager / Vault)
10. App: Settings object
11. (Git repo)
12. (Logs / Sentry)

