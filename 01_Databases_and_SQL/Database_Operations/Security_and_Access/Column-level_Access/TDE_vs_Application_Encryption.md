---
id: "01_Databases_and_SQL/Database_Operations/Security_and_Access/TDE_vs_Application_Encryption"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Database_Operations/Security_and_Access"
tools: ["AWS"]
---

# TDE vs Application Encryption

**L5 Expert** · [↑ Security and Access](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** TDE encrypts the database files and backups transparently so stolen media is useless, while application or client-side encryption keeps specific values unreadable even to the database and its administrators; each solves a different threat.

### 1. The big picture
"Encrypt the database" can mean four quite different things, and each protects against a different attacker:

| Layer | Example | Protects against | Does *not* protect against |
|---|---|---|---|
| Disk/volume | BitLocker, cloud disk encryption | stolen disk | anyone with OS/DB access |
| **TDE** (transparent data encryption) | SQL Server, Oracle, cloud DBs | stolen data files, backup files, detached/attached database | SQL users, DBAs, injection, network sniffing |
| **Column / cell encryption, Always Encrypted** | `ENCRYPTBYKEY`, client-side keys | DBAs and server compromise (Always Encrypted), curious users | query flexibility is reduced |
| **Application-level / tokenization** | encrypt before insert, vault tokens | DB compromise, replicas, backups, logs | bugs in the application, key theft |

The first question, though, is the one Fundamentals of Data Engineering asks: *do you need the sensitive data at all?* Data that is never collected cannot leak; dropping or hashing fields at ingestion beats any encryption scheme.

### 2. Core concepts and vocabulary
- **Encryption at rest / in transit / in use** — protecting stored data, network traffic (TLS), or data while processed.
- **Symmetric encryption (AES)** — one key encrypts and decrypts; fast, suited to bulk and high-write data.
- **Asymmetric encryption (RSA)** — public/private pair; slower, used to protect keys and for signatures, rarely for bulk columns.
- **DEK / KEK, envelope encryption** — a data encryption key encrypts data; a key-encryption key (in a KMS/HSM) encrypts the DEK.
- **TDE** — page-level encryption of data and log files using a database encryption key protected by a certificate or KMS key.
- **Always Encrypted** — client driver encrypts/decrypts; the server only sees ciphertext; keys held outside (column master key in a key vault).
- **Deterministic vs randomized encryption** — deterministic gives equal ciphertext for equal plaintext (equality search, joins possible, patterns leak); randomized does not (safer, not searchable).
- **Tokenization** — replace a value with a random surrogate; the mapping lives in a separate vault.
- **Format-preserving encryption (FPE)** — ciphertext has the same format (16 digits), useful for legacy schemas.
- **Masking** — obscuring display, data itself unchanged.
- **Crypto-shredding** — destroy the key to make all copies of the data unrecoverable.

### 3. How it works, step by step
**TDE**
1. A master key (in master/KMS) protects a certificate or asymmetric key.
2. The certificate protects a database encryption key (DEK) stored in the database boot page.
3. A background scan encrypts every existing page (initial encryption runs online).
4. Pages are encrypted as they are written to disk and decrypted as they are read into memory; query results are plaintext to authorised users.
5. Backups, `tempdb` contents and log files are encrypted; restoring elsewhere needs the certificate/key.

**Client-side (Always Encrypted style)**
1. You define a column master key (CMK) in an external store such as a key vault, and a column encryption key (CEK) wrapped by it.
2. You mark columns as encrypted (deterministic or randomized).
3. The driver encrypts parameters before sending and decrypts results after receiving.
4. The server and DBAs see only ciphertext; operations that need plaintext on the server (functions, wide searches, most joins on randomized columns) are unavailable.

### 4. Worked example
```sql
-- TDE on SQL Server
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<strong>';
CREATE CERTIFICATE TdeCert WITH SUBJECT = 'TDE';
BACKUP CERTIFICATE TdeCert TO FILE='D:\k\TdeCert.cer'
  WITH PRIVATE KEY (FILE='D:\k\TdeCert.pvk', ENCRYPTION BY PASSWORD='<strong2>');

USE Sales;
CREATE DATABASE ENCRYPTION KEY WITH ALGORITHM = AES_256
  ENCRYPTION BY SERVER CERTIFICATE TdeCert;
ALTER DATABASE Sales SET ENCRYPTION ON;

-- progress and state
SELECT db_name(database_id), encryption_state, percent_complete
FROM sys.dm_database_encryption_keys;
```
```sql
-- Always Encrypted column (key objects created beforehand)
CREATE TABLE dbo.Customers (
  CustomerId int PRIMARY KEY,
  CardNumber varchar(19) COLLATE Latin1_General_BIN2
    ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = CEK1,
                    ENCRYPTION_TYPE = DETERMINISTIC,
                    ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256') NOT NULL
);
```
Reading it: after `ENCRYPTION ON`, `encryption_state = 3` means encrypted. **Back up the certificate immediately**; without it, a restore is impossible. For the column, a DBA running `SELECT CardNumber` sees binary ciphertext; only a client holding access to the CMK sees the digits.

### 5. Under the hood
- TDE works at the I/O layer between buffer pool and disk, so there is no change to schema, queries or indexes; overhead is mostly CPU (AES-NI makes it small, often a few percent). It does not shrink compression benefits, but **backup compression** may be less effective on already-encrypted backups unless TDE decrypts before compression is applied (version dependent).
- Because decryption occurs when pages enter memory, any principal who can query sees plaintext: TDE is invisible to authorised or injected SQL.
- **Cell-level encryption** (`ENCRYPTBYKEY`) stores varbinary and loses type, indexes and range predicates; warn: it can bloat storage heavily and hurt performance, so use it sparingly.
- **Whole-value substitution attack**: an attacker with write access copies one encrypted value over another row (e.g. a higher salary or another card). Encryption alone gives confidentiality, not integrity. Defences: bind ciphertext to its row via an authenticator (key-id/row id as authenticator), use authenticated encryption (AEAD), and restrict write permissions.
- **Deterministic** encryption leaks equality and frequency; low-cardinality columns (gender, state) can be reversed by guessing. Choosing it also constrains sharding and indexing: you can index ciphertext for equality only.
- **Key rotation**: for TDE you rotate the wrapping certificate/KMS key (cheap, re-wraps the DEK); rotating the DEK itself re-encrypts all pages online (I/O heavy but not an outage). For envelope schemes rotate the KEK and re-wrap DEKs lazily.

### 6. Performance and design implications
- TDE: set-and-forget, protects backups, satisfies many compliance checklists; measure CPU, log and backup times before/after.
- Application/client encryption: you lose server-side filtering, sorting, joins, aggregates; you decide per column whether searchable (deterministic, blind index/HMAC) or opaque (randomized).
- Tokenization lets joins work on tokens and keeps the real value only in a vault, narrowing audit scope (PCI).
- Keep key custody separate from the DBA team; auditors want to see that no single person can decrypt and exfiltrate.
- Per-tenant keys (BYOK/HYOK) give customers cryptographic revocation; design the key hierarchy before the schema.
- Right to be forgotten across replicas, backups and archives is practical only through crypto-shredding with per-subject keys.
- Post-quantum: for multi-decade retention, "harvest now, decrypt later" argues for crypto-agility (algorithm identifiers, re-encryptable envelopes) even before switching algorithms.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER DATABASE .. SET ENCRYPTION ON` | enable TDE | whole-database at-rest protection |
| `sys.dm_database_encryption_keys` | encryption state/progress | verify and monitor |
| `BACKUP CERTIFICATE` | export TDE cert | immediately after creation |
| `CREATE COLUMN MASTER KEY` / `COLUMN ENCRYPTION KEY` | Always Encrypted keys | client-side column protection |
| `BACKUP ... WITH ENCRYPTION` | encrypted backups | when TDE isn't used or for portability |
| TLS settings (`sslmode`, force encryption) | in-transit protection | always, TDE doesn't cover it |
| `HASHBYTES`/HMAC | hashed or blind-index values | search without reading |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| TDE for compliance and stolen-media risk | the threat is DBAs, injection or replicas |
| Always Encrypted / client-side for card, SSN, health data | you need server-side computation on the column |
| Tokenization for card data used across many systems | you can't run a highly available vault |
| FPE when legacy formats must be preserved | you need strong security guarantees; weaker than random tokens |
| Hashing for lookup-only values | low-entropy values (national IDs) without salt/pepper are brute-forceable |
| Masking for display control | you think it is encryption |

### 9. Common mistakes
- Relying on TDE alone → SQL users, injection and network sniffers still see plaintext → add TLS, least privilege, column protection.
- Losing the certificate/key → backups unrecoverable → export, store off-box, test restores on another instance.
- Keys stored beside the data → encryption is decorative → separate KMS/HSM with distinct custodians.
- Cell encryption everywhere → bloat and slow queries → limit to the few columns that need it.
- Deterministic encryption on low-cardinality columns → pattern analysis → randomized, or don't encrypt.
- Ignoring ciphertext integrity → substitution attacks → authenticated encryption and write controls.
- Encrypting data you did not need → cost and complexity → drop fields at ingestion.
- Masked values mistaken for protection → inference via filters → use encryption/grants for true secrets.

### 10. 🎯 Interview kit
**30-second answer:** TDE is transparent disk-level encryption that defeats stolen files and backups. It does nothing against authorised queries, so for secrets I add column or application-level encryption with keys held outside the DBA domain, plus TLS in transit.

**Structure a longer answer:** clarify the threat (stolen media, insider DBA, app compromise, compliance) → pick layer(s) → key management and rotation → trade-offs in queryability and performance → scenario.

**Likely follow-ups:**
- *What does TDE not protect?* → in-transit data, authorised/injected queries, memory, anyone who can read via SQL.
- *Symmetric vs asymmetric for columns?* → symmetric for throughput; asymmetric to wrap keys.
- *FPE vs tokenization?* → FPE no vault and format-preserving but weaker; tokenization stronger separation, needs vault HA.
- *Rotate TDE keys on multi-TB?* → rewrap DEK with new KEK is metadata; DEK re-encryption runs online and throttled.
- *Hash vs encrypt a searchable national ID?* → hash cannot be reversed but is guessable without salt; encryption reversible and needs key management.

**Signals interviewers look for:** threat-model first, key custody separation, awareness of searchability trade-offs, backup/restore implications.

### 11. Check yourself
- [ ] List three attacks TDE stops and three it does not.
- [ ] Explain envelope encryption and why rotation is cheap.
- [ ] Describe a whole-value substitution attack and a defence.
- [ ] Choose deterministic vs randomized for a column and justify.
- [ ] Sketch crypto-shredding for a per-user erasure request.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [What is the difference between symmetric and asymmetric encryption when used for a column in the database, and which fits high-write throughput?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21407_What_is_the_difference_between_symmetric_and_asymmetric_encr.md>)
- [Deep] [Transparent Data Encryption protects data at rest but what threats does it explicitly NOT protect against?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21376_Transparent_Data_Encryption_protects_data_at_rest_but_what_t.md>)
- [Deep] [What's the difference between column-level encryption and column-level masking and when would masking give a false sense of security?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21388_What_s_the_difference_between_column-level_encryption_and_co.md>)
- [Deep] [How do you secure database backups so a stolen backup file doesn't equal a full data breach?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21439_How_do_you_secure_database_backups_so_a_stolen_backup_file_d.md>)
- [Deep] [Explain the trade-offs of format-preserving encryption versus vault-based tokenization when you must keep card numbers referentially usable across dozens of downstream systems.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21468_Explain_the_trade-offs_of_format-preserving_encryption_versu.md>)
- [Deep] [How would you evaluate whether to adopt post-quantum-ready encryption for data at rest now, given multi-decade data-retention obligations and harvest-now-decrypt-later risk?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21533_How_would_you_evaluate_whether_to_adopt_post-quantum-ready_e.md>)
- [Expert] [How do you handle encryption key rotation for TDE without a full re-encryption outage on a multi-terabyte database?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21396_How_do_you_handle_encryption_key_rotation_for_TDE_without_a_.md>)
- [Expert] [What is your approach to certificate-based mutual TLS between app and database, and how do you manage certificate rotation at scale?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Security_and_Access/Q-21447_What_is_your_approach_to_certificate-based_mutual_TLS_betwee.md>)

## Related concepts

- [Encryption at Rest Strategy](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Encryption_at_Rest_Strategy.md>) *(Encryption and Compliance)*
- [Vendor Compliance Audits](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Vendor_Compliance_Audits.md>) *(Encryption and Compliance)*
- [Secrets and Key Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>) *(Cloud Fundamentals)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › BINARY vs. VARBINARY
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Security Implications
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Encryption of Stored Procedures, Functions, and Views
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Encryption and Data Protection
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Security › 14.6 #99 Exposing the business to whole-value substitution attacks
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Security › 14.6.1 Preparing an encrypted environment
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Undercurrents
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Data ethics, privacy, and compliance
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Security and Privacy › Encryption at rest
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Security Design Patterns › Pattern: Encryptor
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Security Design Patterns › Solution
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Security Design Patterns › Consequences
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Security Design Patterns › Examples
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › Data in Transit
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › Establishing secure data connections
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › Basic connection encryption
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Security › Securely stored secrets
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Applying encryption methods
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Applying encryption methods › Encryption basics
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Additional Interview Questions

