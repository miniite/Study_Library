---
type: "practice"
difficulty: "Intermediate"
---

# What is a checksum or hash and how can it be used to verify a file was not corrupted during transfer?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validating Loads and Deduplication](<../../../../02_Data_Engineering/Data_Quality/Validation/Validating_Loads_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary:

A checksum or hash is a mathematical function that generates a unique identifier for a piece of data, which can be used to ensure the integrity of that data during transmission. In the context of verifying file integrity, we use a checksum to create a hash value before and after transferring a file. If these values match, it indicates that the file has not been corrupted.

#### Triaging the Question

The question is asking for an explanation of what a checksum or hash is and how it can be used in data validation processes, particularly during file transfers. The interviewer might be probing your understanding of cryptographic functions, their application in data integrity checks, and potential use cases and limitations.

#### First-Principles Solution
What is a Checksum/Hash?

A checksum (or more formally a hash) is a function that takes an input (a piece of data or a file) and returns a fixed-size string of characters. This output is the hash value, which uniquely represents the input data.

How Does It Work?
Input Data: Consider a file as our input.
Checksum Calculation: A hashing algorithm processes this file to produce a unique hash value (e.g., SHA-256).
Transmission: This file and its associated hash are sent over the network or stored in a database.
Verification Process:
Receive/Retrieve: On the receiving end, the same hashing function is applied to the received file.
Compare Hashes: If the generated hash value matches the original hash, it indicates that no corruption occurred during transfer or storage.

#### Example
Using SHA-256 (a common algorithm):
PYTHON
import hashlib

# Function to compute SHA-256 hash of a file
def calculate_sha256(file_path):
    sha256_hash = hashlib.sha256()
    with open(file_path, "rb") as f:
        for byte_block in iter(lambda: f.read(4096), b""):
            sha256_hash.update(byte_block)
    return sha256_hash.hexdigest()

# Example usage
original_file_sha = calculate_sha256("example.txt")
print(f"Original file SHA-256: {original_file_sha}")

# After transfer or storage, compute again and compare:
transferred_file_sha = calculate_sha256("received_example.txt")
if original_file_sha == transferred_file_sha:
    print("File is uncorrupted.")
else:
    print("File has been corrupted.")
Why Use Checksums?
Data Integrity: Ensures data remains unchanged during transmission or storage.
Error Detection: Detects accidental changes, such as from network noise or disk corruption.
Security: Protects against malicious tampering by verifying the authenticity of files.

#### Common Mistakes
and Follow-Ups:
Incorrect Hash Function Choice: Using a weaker hash function (e.g., MD5) instead of a stronger one like SHA-256, which is susceptible to collisions.
Not Handling Files Correctly: Failing to read the file correctly, leading to incorrect hash values.
Ignoring Timestamps and Metadata: Not accounting for differences in timestamps or metadata that can change without altering the content.

#### Closing Summary

Checksums are essential tools in ensuring data integrity during transmission and storage. By comparing the hash value of a file before and after transfer, we can reliably detect any corruption or tampering. Understanding how to implement these checks effectively is crucial for maintaining reliable data pipelines.

