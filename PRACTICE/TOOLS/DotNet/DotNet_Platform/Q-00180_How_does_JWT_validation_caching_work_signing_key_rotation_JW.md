---
type: "practice"
difficulty: "Deep"
---

# How does JWT validation caching work (signing key rotation, JWKS refresh) in ASP.NET Core, and how do you avoid a spike of failures when the identity provider rotates keys?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Web APIs and Auth](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Web_APIs_and_Auth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

JWT validation caching is crucial for optimizing performance in web applications while ensuring security by validating tokens against a valid signing key. In ASP.NET Core, JWT validation involves several layers, including caching and handling signing key rotations to maintain robust and efficient operation. When the identity provider rotates keys, it's essential to manage this transition carefully to avoid spikes in failed authentication attempts.

#### Triage

The question is asking about how JWT validation caching works within an ASP.NET Core application, focusing on scenarios where the signing key might rotate, and how these rotations can be managed without causing a spike in token validation failures. The interviewer is probing your understanding of the underlying mechanisms, potential pitfalls, and best practices for maintaining system stability.

#### First-Principles Solution

Understanding JWT Validation Caching:

In ASP.NET Core, Microsoft.AspNetCore.Authentication.JwtBearer middleware handles JWT validation.
By default, it caches validated tokens to improve performance by reusing the same token instance when the same request is made.

Key Rotation and JWKS Refresh:

The signing key used to validate JWTs can rotate over time due to security policies or lifecycle management practices.
JSON Web Key Set (JWKS) contains public keys necessary for validating tokens issued by an identity provider.

Handling Key Rotations:

When the key rotates, the JWKS needs to be refreshed to ensure that new and old signing keys are recognized.
This involves updating the middleware with the latest set of keys without disrupting ongoing validation processes.

Avoiding Spike Failures:

A spike in failures can occur if there's a mismatch between the cached tokens and the updated JWKS, leading to invalid token rejections.
Properly managing the transition period ensures that both old and new keys are recognized during this transitional phase.
Detailed Steps

Initial Setup:

CSHARP
services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(o =>
    {
        o.Authority = "https://your-identity-provider-url.com";
        o.RequireHttpsMetadata = false; // Set to true in production
        o.Audience = "api";
        o.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKeyResolver = (token, securityToken, identifier, parameters) =>
            {
                var keys = GetKeysFromJwks(); // Function to fetch JWKS and extract keys
                return keys.Select(k => new X509SecurityKey((RSA)k)).ToList();
            },
            ValidateIssuer = true,
            ValidIssuer = "https://your-identity-provider-url.com",
            ValidateAudience = true,
            ValidAudience = "api",
            ValidateLifetime = true,
        };
    });

Refreshing JWKS:

Implement a background task to periodically refresh the JWKS from the identity provider.
Use a timer or scheduled tasks like Quartz.NET for this purpose.

Handling Caching During Key Rotation:

Ensure that old and new keys are recognized during the transition period.
Leverage the IssuerSigningKeyResolver to handle multiple keys simultaneously until all tokens have been validated against the latest key.

Graceful Degradation:

Temporarily reduce the cache size or disable caching for a brief period during the key rotation process to prevent any potential issues.
Monitor the system closely and be prepared to revert changes if necessary.

Monitoring and Logging:

Implement logging to capture validation errors and spikes in failure rates.
Use monitoring tools like Application Insights or Prometheus to track performance metrics during key rotations.

#### Closing Summary

JWT validation caching in ASP.NET Core requires careful management of signing key rotations through proper JWKS refresh mechanisms. By ensuring that old and new keys are recognized simultaneously, you can avoid a spike in failures. Key strategies include periodic background tasks for refreshing JWKS, leveraging IssuerSigningKeyResolver, implementing graceful degradation during transitions, and robust monitoring.

#### Common Mistakes/Follow-Ups
Failing to Update JWKS: Not refreshing the JWKS can lead to validation issues.
Ignoring Caching Issues: Disabling or misconfiguring caching can result in performance bottlenecks.
Lack of Monitoring: Without proper logging and monitoring, identifying and addressing issues during key rotations becomes challenging.

If you have any specific concerns about implementing this solution or need further details on a particular aspect, please feel free to ask.

**Code:**

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(o =>
    {
        o.Authority = "https://your-identity-provider-url.com";
        o.RequireHttpsMetadata = false; // Set to true in production
        o.Audience = "api";
        o.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKeyResolver = (token, securityToken, identifier, parameters) =>
            {
                var keys = GetKeysFromJwks(); // Function to fetch JWKS and extract keys
                return keys.Select(k => new X509SecurityKey((RSA)k)).ToList();
            },
            ValidateIssuer = true,
            ValidIssuer = "https://your-identity-provider-url.com",
            ValidateAudience = true,
            ValidAudience = "api",
            ValidateLifetime = true,
        };
    });
}

private List<RSACryptoServiceProvider> GetKeysFromJwks()
{
    // Function to fetch and parse JWKS from identity provider
}
```

**Example input / output:**

```text
// Example of a background task to refresh JWKS
Timer timer = new Timer(async state =>
{
    await Task.Delay(3600 * 1000); // Every hour
    var jwks = GetKeysFromJwks();
    _configuration.GetSection("JwtBearer:TokenValidationParameters:IssuerSigningKeyResolver").Set(jwks);
}, null, TimeSpan.Zero, TimeSpan.FromHours(1));
```

**Flow:**

1. Start
2. Fetch JWKS
3. Parse Keys intoRSACryptoServiceProvider List
4. UpdateTokenValidationParameterswith IssuerSigningKeyResolver
5. Validate JWTs using updatedkeys
6. Monitor Performance andErrors
7. Handle Key RotationsGracefully
8. End

