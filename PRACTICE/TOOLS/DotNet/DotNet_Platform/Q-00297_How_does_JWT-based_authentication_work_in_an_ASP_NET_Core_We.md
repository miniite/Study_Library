---
type: "practice"
difficulty: "Intermediate"
---

# How does JWT-based authentication work in an ASP.NET Core Web API?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Web APIs and Auth](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Web_APIs_and_Auth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

JWT (JSON Web Token) based authentication is a common method used to secure web APIs, particularly in modern microservices architectures. In this context, ASP.NET Core provides robust tools and libraries to implement JWT-based authentication seamlessly.

First, let's summarize the core idea: When a user logs in successfully, an application generates a JWT that contains claims about the user (like their ID, username, roles, etc.). This token is then used by the client for future requests, allowing the server to authenticate without needing to re-authenticate the user each time.

The question is essentially asking how this process works under the hood. The interviewer might be probing your understanding of JWT structure, ASP.NET Core authentication mechanics, and potential security considerations.

To address this from first principles, we'll start by understanding the basic components:

JWT Structure: A JWT consists of three parts: Header, Payload (Claims), and Signature.
Header: Contains the type of token (e.g., "JWT") and the signing algorithm used (e.g., HS256 for HMAC SHA256).
Payload: Includes claims about the user or other information, typically in a JSON format.
Signature: A cryptographic signature ensuring that the JWT has not been tampered with.

In an ASP.NET Core Web API scenario:

User Authentication: When a user logs in successfully, your application generates a JWT using a secret key (usually stored securely).
Token Generation: The JWT contains claims such as sub (subject), name, role, and possibly expiration time (exp).
Client Storage: The client stores the token, typically in local storage or cookies.
Authorization: On subsequent requests, the client sends this token with every request.
Server Verification: The server verifies the JWT using the secret key to ensure its integrity and validity.

Let's walk through a step-by-step example:

Step 1: Setting Up Authentication in ASP.NET Core

First, you need to enable authentication in your ASP.NET Core project. You can do this by adding the Microsoft.AspNetCore.Authentication.JwtBearer package via NuGet.

CSHARP
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;

public void ConfigureServices(IServiceCollection services)
{
    // Add JWT Bearer Authentication
    services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("YOUR_SECRET_KEY")),
            ValidateIssuer = false,
            ValidateAudience = false
        };
    });
}
Step 2: Creating Claims and Generating the JWT

When a user logs in, you can generate a JWT with claims based on their user profile.

CSHARP
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using Microsoft.AspNetCore.Authentication.JwtBearer;

public class TokenService
{
    private readonly string _secretKey = "YOUR_SECRET_KEY";

    public string GenerateToken(User user)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_secretKey);

        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(new Claim[]
            {
                new Claim(ClaimTypes.Name, user.Username),
                new Claim(ClaimTypes.Role, "Admin"),
                // Add more claims as necessary
            }),
            Expires = DateTime.UtcNow.AddDays(7),  // Token expiration after 7 days
            SigningCredentials = new SigningCredentials(
                new SymmetricSecurityKey(key),
                SecurityAlgorithms.HmacSha256Signature)
        };

        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }
}
Step 3: Protecting API Endpoints

You need to protect your API endpoints using the [Authorize] attribute. ASP.NET Core will automatically validate the JWT and ensure that it is valid.

CSHARP
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        // Logic to get product

        return Ok();
    }
}

#### Closing Summary
and Common Mistakes

In summary, JWT-based authentication in ASP.NET Core involves generating a token on the server-side, sending it to the client, and verifying it on each subsequent request. Key considerations include:

Security: Ensure that your secret key is securely managed.
Expiry: Set appropriate expiration times for tokens to minimize risk if they are compromised.
Audience and Issuer Validation: If using multiple domains or services, validate these fields.

#### Common mistakes
include:

Not securing the secret key properly.
Failing to invalidate tokens upon logout or password changes.
Ignoring token expiry, leading to potential security risks.

**Comparison:**

| Component  | Description |
|------------|-------------|
| Header     | Type and Signing Algorithm |
| Payload    | User Claims (e.g., username, roles) |
| Signature  | Ensures Token Integrity |

**Code:**

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;

public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("YOUR_SECRET_KEY")),
            ValidateIssuer = false,
            ValidateAudience = false
        };
    });
}

public class TokenService
{
    private readonly string _secretKey = "YOUR_SECRET_KEY";

    public string GenerateToken(User user)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_secretKey);

        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(new Claim[]
            {
                new Claim(ClaimTypes.Name, user.Username),
                new Claim(ClaimTypes.Role, "Admin"),
            }),
            Expires = DateTime.UtcNow.AddDays(7),  // Token expiration after 7 days
            SigningCredentials = new SigningCredentials(
                new SymmetricSecurityKey(key),
                SecurityAlgorithms.HmacSha256Signature)
        };

        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }
}
```

**Example input / output:**

```text
public IActionResult Login(UserLoginRequest request)
{
    // Logic to verify user credentials

    var token = TokenService.GenerateToken(user);
    return Ok(new { token });
}

[HttpGet("{id}")]
public IActionResult GetProduct(int id, [FromServices] TokenService tokenService)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    // Logic to get product

    return Ok();
}
```

**Data:**

```json
{
  "token": "<YOUR_JWT_TOKEN>",
  "user": {
    "username": "JohnDoe",
    "role": "Admin"
  }
}
```

**Flow:**

1. User Logs In
2. Generate JWT
3. Send JWT to Client
4. Client Stores JWT
5. Make API Request with JWT
6. Verify JWT on Server
7. Access Allowed / AccessDenied

