---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd implement GDPR pseudonymization versus anonymization in a data warehouse feeding analytics — where does re-identification risk creep back and how do you govern the mapping table?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Compliant Access to Encrypted PII](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Compliant_Access_to_Encrypted_PII.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address GDPR pseudonymization versus anonymization in a data warehouse feeding analytics, we need to understand that both techniques aim to reduce re-identification risks but differ significantly in their approach and implications. Pseudonymization replaces identifying information with artificial identifiers (pseudonyms) while retaining the ability to re-identify individuals if necessary. Anonymization, on the other hand, irreversibly eliminates any links between data points and personal identifiers. Here’s a deep dive into implementing these techniques, focusing specifically on pseudonymization in a GDPR-compliant context.

Firstly, let's establish the core requirements for pseudonymization:

Data Integrity: The pseudonymized data must still be useful for analytics.
Re-identification Risk Management: Sensitive information should not be easily re-identifiable.
Governance and Traceability: Detailed documentation of the mapping between pseudonyms and real identities is crucial.

#### Triaging the Question

The interviewer is probing your understanding of:

The definitions and differences between pseudonymization and anonymization.
How to implement pseudonymization in a data warehouse.
Potential re-identification risks and how to mitigate them.
Governance frameworks for mapping tables that preserve pseudonymized data.

#### First-Principles Solution
Step 1: Define Pseudonymization Techniques

Pseudonymization can be implemented using various methods such as:

Tokenization: Replacing sensitive fields with tokens.
Hashing: Using cryptographic functions to transform the data into a fixed-length string.
Shuffling and Masking: Randomly shuffling or masking parts of the data.
Step 2: Implement Data Warehouse Design

Design the data warehouse schema with pseudonymized data storage:

Pseudonymization Layer: Store the original sensitive data separately but securely, using strong encryption techniques like AES-256.
Mapping Table: Create a mapping table that links each pseudonym to its real identity. This table should be stored in a highly secured environment with restricted access.
Step 3: Re-identification Risk Mitigation

To ensure re-identification risk does not creep back, follow these best practices:

Access Controls: Implement strict access controls on the mapping table.
Data Masking: Use data masking techniques to hide sensitive parts of the pseudonymized data in the analytics environment.
Audit Logs: Maintain audit logs for any access or use of the mapping table.
Step 4: Governance and Compliance

Ensure compliance with GDPR by:

Documentation: Keeping detailed documentation on how, where, and why pseudonymization is used.
Regular Audits: Conduct regular audits to ensure adherence to security policies and regulatory requirements.
Data Subject Rights: Implement mechanisms for data subjects to exercise their rights under GDPR, such as requesting the deletion of their personal data.

#### Closing Summary

Pseudonymization in a data warehouse requires careful design and strict governance to mitigate re-identification risks while still enabling useful analytics. By implementing proper access controls, data masking techniques, and robust audit processes, we can ensure that the data remains compliant with GDPR standards.

#### Common Mistakes/Follow-Ups
Ignoring Re-Identification Risks: Failing to recognize the potential for re-identification through clever data analysis.
Lack of Access Controls: Not implementing strict access controls on sensitive mapping tables.
Insufficient Documentation: Poorly documenting the pseudonymization process, making it difficult to audit and maintain compliance.

**Code:**

```sql
-- Example SQL for creating a pseudonymized data warehouse schema
 CREATE TABLE sales_data (
    sale_id INT PRIMARY KEY,
    customer_pseudonym VARCHAR(255),
    product_id INT,
    amount DECIMAL(10, 2),
    purchase_date TIMESTAMP
 );

 -- Mapping table example
 CREATE TABLE customer_mapping (
    pseudonym_id INT PRIMARY KEY,
    real_customer_id INT NOT NULL,
    mapping_key VARCHAR(36) NOT NULL CHECK (LENGTH(mapping_key) = 36)
 );
```

**Example input / output:**

```text
Input:
 - Sales data table with customer pseudonyms and original IDs.
 - Mapping table linking pseudonyms to real IDs.

 Output:
 - Pseudonymized sales data for analytics, unusable without the mapping table.
 - Audit logs showing access attempts to the mapping table.
```

**Data:**

```json
{
   "pseudonymizationLayer": {
     "sales_data_table": [
       { "sale_id": 1, "customer_pseudonym": "P001", "product_id": 2, "amount": 59.99, "purchase_date": "2023-04-01" },
       { "sale_id": 2, "customer_pseudonym": "P002", "product_id": 3, "amount": 79.99, "purchase_date": "2023-04-02" }
     ],
     "mapping_table": [
       { "pseudonym_id": 1, "real_customer_id": 123456, "mapping_key": "e3b0c44298fc..." },
       { "pseudonym_id": 2, "real_customer_id": 789012, "mapping_key": "d41d8cd98f00..." }
     ]
   }
 }
```

