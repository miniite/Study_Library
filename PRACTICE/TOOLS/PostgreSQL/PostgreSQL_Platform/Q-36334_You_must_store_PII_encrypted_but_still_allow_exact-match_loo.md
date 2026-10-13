---
type: "practice"
difficulty: "Deep"
---

# You must store PII encrypted but still allow exact-match lookup on email; explain deterministic encryption via a blind index (HMAC column) versus pgcrypto, and the security trade-off of deterministic ciphertext.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ UUID vs Serial Keys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/UUID_vs_Serial_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement in PostgreSQL, we need to balance data privacy with efficient query capabilities. Deterministic encryption can be used to store PII (Personal Identifiable Information) such as email addresses securely while allowing exact-match lookups using a blind index based on HMAC or leveraging pgcrypto functions. Let's explore both approaches and the security trade-offs involved.

The core idea is to ensure that the encrypted data remains searchable, but it should also maintain confidentiality by preventing unauthorized access. This involves encrypting the email in such a way that we can still perform exact lookups while ensuring the ciphertexts are unpredictable and non-reversible without the correct key.

Firstly, let's consider deterministic encryption via HMAC (Hash-based Message Authentication Code). HMAC is a form of hash function that incorporates a secret key to generate a unique digest. This method allows us to create a fixed-length output from an input but does not provide full cryptographic security like encryption algorithms do. Here’s how we can implement it:

SQL
-- Create table with encrypted email using HMAC
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT NOT NULL
);

-- Function to generate HMAC for email
CREATE OR REPLACE FUNCTION gen_hmac(email TEXT, key BYTEA) RETURNS BYTEA AS $$
SELECT.digest(
    hmac($1::bytea, $2)
);
$$ LANGUAGE sql;

-- Example usage: Encrypting an email with a secret key
DO $$
DECLARE
    key BYTEA := 'secret_key';
    encrypted_email BYTEA;
BEGIN
    INSERT INTO users (email) VALUES ('user@example.com');
    SELECT gen_hmac('user@example.com', key) INTO encrypted_email;
END $$;

-- Index on HMAC to enable efficient lookups
CREATE INDEX idx_users_hmac ON users ((gen_hmac(email, key)));

On the other hand, we can use pgcrypto for deterministic encryption. pgcrypto provides various functions like encrypt, which can be used to encrypt data deterministically. However, this approach requires careful handling of the encryption and decryption keys.

SQL
-- Install pgcrypto if not already installed
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Function to encrypt email using deterministic encryption
CREATE OR REPLACE FUNCTION encrypt_email(email TEXT) RETURNS BYTEA AS $$
SELECT encrypt($1::text, 'mykey', 'aes-256-cbc');
$$ LANGUAGE sql;

-- Example usage: Encrypting an email with a secret key
DO $$
DECLARE
    encrypted_email BYTEA;
BEGIN
    INSERT INTO users (email) VALUES ('user@example.com');
    SELECT encrypt_email('user@example.com') INTO encrypted_email;
END $$;

-- Index on the encrypted column to enable efficient lookups
CREATE INDEX idx_users_encrypted ON users ((encrypt_email(email)));

#### Triaging the Question

The interviewer is probing your understanding of deterministic encryption, specifically in the context of PostgreSQL. They are likely interested in:

Deterministic vs. Non-deterministic Encryption: Deterministic encryption generates the same ciphertext for the same plaintext under a given key. This can be useful for exact-match lookups but has security implications.
Security Trade-offs: The choice between HMAC and pgcrypto involves balancing performance, security, and confidentiality.
Security Considerations
HMAC: Provides integrity but not confidentiality. If an attacker knows the plaintext or can guess it, they can potentially determine if a given email exists in the database.
pgcrypto: Offers stronger encryption but requires careful key management and storage to prevent exposure of the encryption keys.

#### Closing Summary

Both HMAC and pgcrypto have their place in deterministic encryption for PostgreSQL. HMAC is simpler and more performant but sacrifices some security, while pgcrypto provides better confidentiality at the cost of complexity. The choice depends on the specific security requirements and performance constraints of your application.

#### Common Mistakes

Overlooking Key Management: For pgcrypto, failing to securely manage encryption keys can compromise the entire system.
Assuming HMAC Ensures Confidentiality: HMAC only ensures integrity, not confidentiality. An attacker who knows or guesses the plaintext can still determine if it exists in the database.

**Comparison:**

| email               |
|---------------------|
| user@example.com    |
| encrypted_email     |
|---------------------|
| 0x987a34f5b2d1...   |

**Code:**

```sql
-- Deterministic Encryption via HMAC
CREATE OR REPLACE FUNCTION gen_hmac(email TEXT, key BYTEA) RETURNS BYTEA AS $$
SELECT.digest(
    hmac($1::bytea, $2)
);
$$ LANGUAGE sql;

CREATE INDEX idx_users_hmac ON users ((gen_hmac(email, key)));

-- Encrypt Email using pgcrypto
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE OR REPLACE FUNCTION encrypt_email(email TEXT) RETURNS BYTEA AS $$
SELECT encrypt($1::text, 'mykey', 'aes-256-cbc');
$$ LANGUAGE sql;

CREATE INDEX idx_users_encrypted ON users ((encrypt_email(email)));
```

**Example input / output:**

```

-- Input for HMAC example DO $$ DECLARE key BYTEA := 'secret_key'; BEGIN INSERT INTO users (email) VALUES ('user@example.com'); SELECT gen_hmac('user@example.com', key); END $$;

-- Output for HMAC example hmac_value | 0x2b58d0f79e34a1b4...

CODE
Copy

```sql
-- Input for pgcrypto example
DO $$
DECLARE
BEGIN
    INSERT INTO users (email) VALUES ('user@example.com');
    SELECT encrypt('user@example.com', 'mykey', 'aes-256-cbc');
END $$;

-- Output for pgcrypto example
encrypted_email | 0x987a34f5b2d1...
```

**Data:**

```sql
-- Example data for HMAC and pgcrypto
INSERT INTO users (email) VALUES ('user@example.com');

SELECT gen_hmac('user@example.com', 'secret_key') AS hmac_value;
SELECT encrypt('user@example.com', 'mykey', 'aes-256-cbc') AS encrypted_email;
```

**Flow:**

1. Yes
2. No
3. Generate HMAC
4. Is Integrity Only Sufficient?
5. HMAC for Lookup
6. Use pgcrypto for DeterministicEncryption
7. Index on HMAC Value
8. Index on Encrypted Email

