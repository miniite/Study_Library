---
type: "practice"
difficulty: "Intermediate"
---

# What is an SSL/TLS certificate and how does a browser decide to trust it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ mTLS and Zero Trust](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/mTLS_and_Zero_Trust.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll explore what an SSL/TLS certificate is and how a web browser decides to trust one. We'll start with a brief overview of SSL/TLS certificates and then dive into their role in securing internet communications.

TL;DR

An SSL/TLS certificate is a digital document that ensures the authenticity and security of websites by verifying the identity of the server hosting the site. Browsers use these certificates to establish secure connections, relying on trust anchors (certification authorities) to validate the certificate's legitimacy. When you visit an HTTPS website, the browser performs several checks to ensure it trusts the certificate.

#### Triaging the Question

The question asks two key aspects:

What is an SSL/TLS certificate?
How does a browser decide to trust it?

We'll address both parts step by step from first principles.

Understanding SSL/TLS Certificates

An SSL (Secure Sockets Layer) or TLS (Transport Layer Security) certificate is a digital document that binds a cryptographic key with the domain name of the website. It's used to establish an encrypted link between a web server and a browser. The primary purpose of an SSL/TLS certificate is to ensure data encryption, verify the identity of the site owner, and protect sensitive information such as credit card details.

Key Components:
Domain Name: Identifies the specific domain name for which the certificate is issued.
Public Key: Used by browsers to encrypt data sent to the server.
Private Key: Kept secret on the server side; it’s used to decrypt data sent from the browser.
Certificate Authority (CA): An entity that issues and manages certificates, ensuring their authenticity.
How Browsers Decide to Trust an SSL/TLS Certificate

When you access a website over HTTPS, your browser performs several steps to verify the certificate's legitimacy:

Requesting the Certificate: The client sends a request for the server’s public key.
Receiving and Verifying the Certificate:
Browsers use a list of trusted CAs (often provided by the operating system) to validate the certificate.
The browser checks if the certificate is signed by one of these trusted CAs.
Certificate Chain Validation: Browsers verify that all intermediate certificates in the chain are valid and issued by trusted authorities, ultimately linking back to a root CA.
Expiry and Revocation Checks: Browsers check the date range during which the certificate is valid and whether it has been revoked.
Step-by-Step Explanation

Client Request:

The browser initiates a connection with the server over HTTPS.
It sends a Server Name Indication (SNI) message, specifying the domain name to the server.

Certificate Exchange:

The server responds with its SSL/TLS certificate and public key.
If the client trusts the CA that issued this certificate, it proceeds; otherwise, an error is reported.

Trust Validation:

Browsers maintain a list of trusted CAs stored in their security databases.
They verify if the certificate was signed by one of these CAs.

Certificate Chain Verification:

If the certificate has a chain (often necessary for multi-level trust), each subsequent certificate in the chain is validated similarly.
The verification process continues until it reaches a root CA, which is considered trusted and included in the browser’s trust store.

Expiry and Revocation Checks:

Browsers check if the current date falls within the valid start and end dates of the certificate.
They also query revocation lists (CRLs) or use Online Certificate Status Protocol (OCSP) to ensure the certificate has not been revoked.

#### Common Mistakes
and Follow-Ups
Assuming All Browsers Trust the Same CAs: While most modern browsers trust a core set of CAs, there can be differences. For example, some organizations might replace or add their own trusted CA.
Ignoring Certificate Chains: A certificate that isn’t part of an expected chain can lead to warnings or errors.
Failing to Check Expiry Dates: Certificates with expired dates can result in connection errors.

#### Closing Summary

Understanding SSL/TLS certificates and how browsers trust them is crucial for ensuring secure web communications. Browsers perform a series of checks, starting from certificate validation by trusted CAs to expiry date and revocation status, to establish a secure connection over HTTPS.

**Code:**

```python
# Example Python code snippet to check SSL/TLS certificate validity using OpenSSL library

from OpenSSL import crypto

def validate_certificate(cert_file_path):
    with open(cert_file_path, "rt") as cert_file:
        cert_data = cert_file.read()

    x509 = crypto.load_certificate(crypto.FILETYPE_PEM, cert_data)
    subject = x509.get_subject().CN
    issuer = x509.get_issuer().CN

    print(f"Subject: {subject}")
    print(f"Issuer: {issuer}")

validate_certificate("path/to/certificate.pem")
```

**Example input / output:**

```text
Request:
- Client sends an HTTPS request to a server.

Response (for a valid certificate):
- Server responds with its SSL/TLS certificate.
- Browser verifies the certificate using trusted CAs, certificate chains, and expiry dates.

Response (for an invalid certificate):
- Browser displays a warning message indicating that the site’s security certificate is not trusted.
```

**Flow:**

1. Yes
2. No
3. Client initiates HTTPS request
4. Browser requests server'spublic key and certificate
5. Server sends its certificate
6. Browser checks if thecertificate is signed by atrusted CA
7. Certificate valid?
8. Browser establishes secureconnection
9. Browser displays error orwarning message
10. Secure HTTPS connectionestablished

