---
type: "practice"
difficulty: "Deep"
---

# Explain the trade-offs of format-preserving encryption versus vault-based tokenization when you must keep card numbers referentially usable across dozens of downstream systems.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ TDE vs Application Encryption](<../../../../01_Databases_and_SQL/Database_Operations/Security_and_Access/Column-level_Access/TDE_vs_Application_Encryption.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with sensitive data like credit card numbers, ensuring both security and usability is critical. In a scenario where we need to maintain referential integrity across multiple downstream systems, two common approaches are format-preserving encryption (FPE) and vault-based tokenization. Each approach has its trade-offs that must be carefully considered.

#### Triage

The question asks us to weigh the pros and cons of FPE versus vault-based tokenization when we need to keep card numbers referentially usable across multiple downstream systems. The interviewer is probing our understanding of security, usability, performance, complexity, and maintenance costs associated with these methods.

#### Assumptions
Number of Systems: We are working with dozens of downstream systems.
Data Sensitivity: Credit card numbers are highly sensitive.
Usability Requirement: The system must maintain referential integrity to ensure that data can be used across different applications and services without direct access to the plaintext card number.

#### First-Principles Solution
Format-Preserving Encryption (FPE)

#### Overview
Format-preserving encryption allows us to encrypt a sensitive piece of information such as a credit card number while preserving its format. This means that the encrypted value still appears to be a valid credit card number but is unreadable without the decryption key.

Key Features and Trade-offs:

Security: FPE offers strong security by ensuring that the ciphertext looks like the original plaintext, making it difficult for attackers to discern the data.
Usability: Cards can remain referentially usable within the system because they retain their format. This ensures seamless integration with downstream systems without requiring any changes in their logic.
Performance: FPE generally has a lower computational overhead compared to other encryption methods, as it doesn't require converting the data into an unreadable form and back.
Complexity: Implementing FPE can be complex due to its reliance on specific cryptographic algorithms that must preserve the format of the plaintext.
Vault-Based Tokenization

#### Overview
Tokenization involves replacing sensitive data with non-sensitive placeholders (tokens) stored in a vault, which manages the mapping between tokens and their original values. This ensures that no one can see or use the actual card number directly.

Key Features and Trade-offs:

Security: Tokenization provides strong security by decoupling the token from the sensitive data, making it difficult for attackers to gain access.
Usability: Usability is more challenging with tokenization because downstream systems need to be modified to use tokens instead of card numbers. This can involve significant changes in application logic and may require custom development efforts.
Performance: Tokenization has higher overhead due to the additional steps required for token generation, storage, and lookup in the vault.
Complexity: Managing a token vault introduces additional complexity, as it requires robust management of tokens, mapping, and security policies.

#### Closing Summary

Both FPE and vault-based tokenization have their merits when dealing with sensitive data like credit card numbers. FPE is more straightforward for maintaining referential integrity but may be less secure due to its format-preserving nature. Vault-based tokenization offers better security by decoupling the data, making it harder to access, but can introduce significant complexity and performance overhead.

#### Common Mistakes
and Follow-Ups
Overlooking Security: A common mistake is assuming that FPE provides sufficient security because of its format preservation.
Underestimating Complexity: Sometimes, developers underestimate the complexity involved in maintaining a token vault, leading to poor data management practices.
Neglecting Performance: Ignoring the performance impact of FPE or tokenization can lead to suboptimal system design.

To determine which approach is better, we should consider factors such as the specific security requirements, the number and nature of downstream systems, and any existing constraints in the technical infrastructure. A comprehensive risk assessment and a thorough understanding of both methods are crucial for making an informed decision.

**Comparison:**

```
Data Format	FPE (Format-Preserving Encryption)	Tokenization
Security	Good, format preserved	Better, data decoupled
Usability	Easy to maintain referential integrity	Requires downstream system modifications
Performance	Lower overhead	Higher overhead due to token management
Complexity	Medium complexity	High complexity with token vault management
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Downstream System 1] --> B[Uses Encrypted/Tokenized Data];
    C[Downstream System 2] --> D[Uses Encrypted/Tokenized Data];
    E[Database] --> F[Manages Encryption/Tokenization];
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
Original Credit Card Number: 4111-1111-1111-1111
FPE Encrypted Value: 4111-1111-1111-1111 (Same format)
Tokenized Value: TTT-TTT-TTT-AAA (Different format)

Output:

PLAINTEXT
Copy
FPE Encrypted Value: 4111-1111-1111-1111
Tokenized Value: TTT-TTT-TTT-AAA
```

**Data:**

```json
{
    "creditCardNumbers": [
        {"cardNumber": "4111-1111-1111-1111", "encryptedValue": "4111-1111-1111-1111"},
        {"cardNumber": "5105-1051-0510-6", "tokenizedValue": "TTT-TTT-TTT-AAA"}
    ]
}
```

**Flow:**

1. Data Entry
2. Encrypt/Tokenize
3. FPE Algorithm or Vault System
4. Storing Encrypted/TokenizedData
5. Querying DownstreamSystems
6. Decrypt/Tokenize Before Use

