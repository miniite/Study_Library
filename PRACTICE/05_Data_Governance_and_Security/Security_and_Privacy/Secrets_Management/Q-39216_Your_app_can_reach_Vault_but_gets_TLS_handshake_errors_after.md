---
type: "practice"
difficulty: "Intermediate"
---

# Your app can reach Vault but gets TLS handshake errors after a cert rotation. How do you debug the trust chain?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rotation with Dynamic and Static Secrets](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Rotation_with_Dynamic_and_Static_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that our application correctly trusts the new certificate issued by Vault following a recent certificate rotation. The primary challenge is debugging TLS handshakes in an environment where the certificate has changed but the application might still be holding onto old trust material.

To debug and resolve this issue, let's break it down step-by-step from first principles:

Verify the New Certificate Chain: First, confirm that the new certificates are installed correctly on Vault and are accessible.
Inspect Trust Store Configuration: Check how your application is configured to trust certificates.
Update Application Trust Store: Ensure that the application’s trust store has been updated with the new certificate(s).
Validate TLS Handshake: Test the application’s ability to perform a successful TLS handshake using the updated certificates.

Let's go through these steps in detail:

Step-by-Step Solution
1. Verify the New Certificate Chain

First, confirm that the new certificates are correctly installed and accessible on Vault. Use tools like openssl or curl -v to verify the certificate chain:

SH
$ openssl s_client -connect <vault-ip>:8200 </path/to/cert.pem

This command will display details about the SSL connection, including any issues with the certificate chain.

2. Inspect Trust Store Configuration

Determine how your application configures its trust store. Commonly, this is done via a configuration file or environment variables:

Java: truststore and keystore configurations in JVM arguments.
Python: Using requests library with custom certificate authorities.
Go: Configuring TLSConfig.
3. Update Application Trust Store

Ensure that the application’s trust store is updated with the new certificates:

SH
# Example for Java: Update JVM argument
java -Djavax.net.ssl.trustStore=/path/to/new/truststore.jks -jar myapp.jar

# For Go, update TLSConfig:
import (
    "crypto/tls"
)

func main() {
    config := &tls.Config{RootCAs: myCertPool}
    conn, err := tls.Dial("tcp", "<vault-ip>:8200", config)
    if err != nil {
        panic(err)
    }
    // ... rest of the code
}
4. Validate TLS Handshake

After updating the trust store, test the application’s ability to perform a successful TLS handshake:

SH
$ curl -v --cacert /path/to/new/cert.pem https://<vault-ip>:8200

This command will help verify if the updated certificate is being correctly trusted.

#### Common Mistakes
and Follow-Ups

Mistake: Not properly updating the application’s trust store.

#### Follow-Up
Ensure that all relevant components, including any proxies or load balancers, also have their trust stores updated.

Mistake: Ignoring intermediate certificates in the chain.

#### Follow-Up
Verify that the entire certificate chain is correctly installed and trusted.

#### Closing Summary

By systematically verifying each component of the TLS handshake process, we can effectively debug and resolve issues arising from a certificate rotation. Ensuring thorough updates and validation across all layers will minimize disruptions to application reliability.

**Comparison:**

```
Step	Action
1	Verify new certificates are installed on Vault.
2	Inspect the application's trust store configuration.
3	Update the application’s trust store with the new certificate(s).
4	Test TLS handshake to ensure successful connection.
```

**Code:**

```java
// Example Java JVM argument update
java -Djavax.net.ssl.trustStore=/path/to/new/truststore.jks -jar myapp.jar

// Go TLSConfig example
import (
    "crypto/tls"
)

func main() {
    caCert, err := tls.LoadX509KeyPair("/path/to/new/cert.pem", "/path/to/private.key")
    if err != nil {
        panic(err)
    }
    rootCAs := x509.NewCertPool()
    rootCAs.AddCert(caCert.Certificate[0])
    config := &tls.Config{RootCAs: rootCAs}
    conn, err := tls.Dial("tcp", "<vault-ip>:8200", config)
    if err != nil {
        panic(err)
    }
    // ... rest of the code
}
```

**Example input / output:**

```text
$ openssl s_client -connect <vault-ip>:8200 </path/to/new/cert.pem
---
Certificate chain
 0 s:/CN=vault.example.com
   i:/CN=MyCertAuthority
---
```

**Data:**

```json
{
  "certificate_chain": [
    "/path/to/new/cert.pem",
    "/path/to/intermediate-cert.pem",
    "/path/to/root-ca.pem"
  ]
}
```

**Flow:**

1. Verify New Certificates onVault
2. Inspect Trust Store
3. Update Application's TrustStore
4. Test TLS Handshake

