---
type: "practice"
difficulty: "Expert"
---

# Your PKI engine issued a cert with a 90-day TTL but you need to revoke it now. Walk through revocation and CRL/OCSP implications.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rotation with Dynamic and Static Secrets](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Rotation_with_Dynamic_and_Static_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the scenario where a certificate issued by your PKI engine has a 90-day TTL but needs immediate revocation, we need to walk through the process of certificate revocation and understand its implications on Certificate Revocation Lists (CRL) and Online Certificate Status Protocol (OCSP).

TL;DR

Revoking a certificate involves marking it as invalid in the PKI infrastructure. This is done by adding an entry to the CRL or updating the OCSP responder. The revocation status needs to be communicated promptly to all relevant entities, including clients, intermediate CAs, and any other systems relying on this certificate.

#### Triaging the Question

The question revolves around how to handle immediate revocation of a PKI-issued certificate with a 90-day TTL that has already been issued. The key elements to consider are:

Revocation Process: Marking the certificate as invalid.
CRL (Certificate Revocation List): A periodic list of revoked certificates.
OCSP (Online Certificate Status Protocol): An online method for checking a certificate's revocation status.

#### Assumptions
and Context
The PKI engine is configured with both CRL and OCSP mechanisms.
The certificate in question has been issued but needs to be revoked urgently.
The PKI infrastructure supports dynamic updates (for example, via CRL publishing or OCSP).
Step-by-Step Solution

Identify the Revocation Reason:

Determine why the certificate is being revoked. Common reasons include security breaches, key compromise, or policy violations.

Update the Certificate Status in the PKI Engine:

Immediately mark the certificate as revoked within the PKI engine.
Ensure that any relevant key material (e.g., private keys) are securely destroyed and cannot be used again.

Generate a CRL Entry:

If using CRL, add an entry for the revoked certificate to the next scheduled CRL update.
The CRL should include details such as the reason code, revocation time, and the certificate serial number.

Publish the Updated CRL:

Distribute the updated CRL across all distribution points (e.g., via HTTP or FTP).
Ensure that clients and systems can download and refresh their CRLs to reflect the new information.

Update OCSP Responder:

If using OCSP, update the responder with the revocation status.
The OCSP responder should be able to return the correct revocation status when queried for this certificate.

Notify Dependent Systems and Clients:

Notify relevant systems and clients about the immediate revocation of the certificate.
Ensure that any ongoing processes using this certificate are stopped or redirected.

Monitor and Validate Revocation Status:

Verify that all systems have updated their CRLs or OCSP caches to reflect the new status.
Monitor for any lingering issues and ensure that no unauthorized access occurs due to the revoked certificate.

#### Closing Summary
and Common Mistakes

Revoking a certificate involves multiple steps, from marking it as invalid in the PKI engine to distributing updates via CRL and OCSP. Key points include:

Immediate Action: Mark the certificate as revoked promptly.
Correct Distribution: Ensure that all relevant entities (CRL distribution points and OCSP responders) are updated with the new status.
Notification: Inform dependent systems of the revocation.

#### Common Mistakes
Delayed Action: Failing to revoke the certificate immediately can leave it in use, potentially leading to security breaches.
Incorrect CRL Distribution: Not updating all distribution points or failing to notify relevant clients can result in continued use of a revoked certificate.
Insufficient Monitoring: Not monitoring for lingering issues can lead to undetected misuse of the certificate.

**Comparison:**

```
Step	Action	Responsible Party
1	Mark as revoked in PKI Engine	PKI Administrator
2	Generate CRL Entry	PKI Administrator
3	Publish CRL	PKI Administrator
4	Update OCSP Responder	PKI Administrator
5	Notify Dependent Systems	System Operator
```

**Code:**

```python
# Example Python code snippet to revoke a certificate via an API
import requests

def revoke_certificate(certificate_id, reason_code):
    url = "https://pki.example.com/api/certificates/{certificate_id}/revoke"
    payload = {"reasonCode": reason_code}
    response = requests.post(url, json=payload)
    return response.status_code == 200
```

**Example input / output:**

```text
Input:
Request to revoke certificate with ID "ABC123".

Output:
Status code 200 indicates successful revocation.
```

**Data:**

```json
{
  "CRL_Entry": {
    "SerialNumber": "123456789ABCDEF",
    "RevocationReason": "keyCompromise",
    "RevokedAt": "2023-10-01T12:00:00Z"
  }
}
```

**Flow:**

1. Mark Certificate as Revoked
2. Generate CRL Entry
3. Publish CRL
4. Update OCSP Responder
5. Notify Dependent Systems
6. Monitor and ValidateRevocation Status

