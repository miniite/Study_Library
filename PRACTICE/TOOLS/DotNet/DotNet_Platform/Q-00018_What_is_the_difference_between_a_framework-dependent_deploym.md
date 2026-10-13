---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a framework-dependent deployment and a self-contained deployment when publishing a .NET app?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ JIT vs AOT Compilation](<../../../../TOOLS/DotNet/DotNet_Platform/JIT_vs_AOT_Compilation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a backend software development context, especially with .NET applications, understanding the differences between framework-dependent (FD) and self-contained (SC) deployments is crucial. In summary, FD deployments rely on having specific versions of the .NET runtime installed on the target machine, while SC deployments include all necessary runtime components within the application package itself.

The question is probing your knowledge of deployment models in .NET and how they impact the distribution and execution environment of an application. Here’s a detailed breakdown to clarify these concepts:

#### Triage
Framework-Dependent Deployment: This model assumes that the target machine already has the required version of the .NET runtime installed.
Self-Contained Deployment: This approach includes all necessary dependencies within the application package, ensuring it can run independently of other software installations.

#### Assumptions
The discussion is centered around deploying a .NET application on different machines without requiring additional setup or configuration.
You are familiar with .NET Core and .NET Framework (though .NET 6+ often uses the term "self-contained" more broadly).

#### First-Principles Solution
Framework-Dependent Deployment
Runtime Requirement: The target machine must have the appropriate version of the .NET runtime installed.
Advantages:
Smaller package size because only application code is packaged.
Faster deployment since no additional files need to be downloaded.
Disadvantages:
Security risk if a different version of the runtime is required, and that version has vulnerabilities not yet patched in your environment.
Compatibility issues if the target machine has an incompatible .NET version.
Self-Contained Deployment
Independent Execution: The application includes its own copy of the runtime library, ensuring compatibility and security.
Advantages:
No external dependencies required; the app runs seamlessly even on machines without a compatible .NET installation.
Easier deployment as it doesn’t depend on other software installations.
Disadvantages:
Larger package size due to including the runtime library and additional files.
Slower initial download time.

#### Closing Summary

In summary, FD deployments are ideal for environments where you have control over the target machine's .NET version, such as in development or testing scenarios. SC deployments provide a more robust and independent solution, suitable for production environments where ensuring compatibility and security is crucial.

#### Common Mistakes/Follow-Ups
Misunderstanding Compatibility: FD might lead to issues if there are newer updates that introduce breaking changes.
Deployment Size Overhead: SC deployments can significantly increase the package size, which may not be ideal in constrained environments like mobile devices or IoT applications.
Environment Setup: Ensure you test both deployment models thoroughly before choosing one.

For a more nuanced understanding, consider scenarios such as deploying to different operating systems, managing dependencies through NuGet, and evaluating long-term maintenance costs. Understanding these trade-offs will help you make informed decisions based on the specific needs of your project.

**Comparison:**

```
Deployment Model	Requires Framework	Package Size	Ease of Deployment
Framework-Dependent	Yes	Smaller	Faster
Self-Contained	No	Larger	Easier
```

**Code:**

```csharp
// Example: Publishing settings in .NET Core CLI
dotnet publish -c Release -r win-x64 --self-contained false # Framework-dependent
dotnet publish -c Release -r win-x64 --self-contained true  # Self-contained
```

**Example input / output:**

```
Input: dotnet publish -c Release -r win-x64 --self-contained false
Output:
PLAINTEXT
Copy
Project XYZ (.NETCoreApp,Version=v3.1) was compiled using framework .NET Core 3.1.0 (aspnetcore-runtime-3.1.2).
  Microsoft.NETCore.App 3.1.2 [C:\Program Files\dotnet\shared\Microsoft.NETCore.App]
  Microsoft.WindowsDesktop.App 3.1.2 [C:\Program Files\dotnet\shared\Microsoft.WindowsDesktop.App]

Publishing to C:\path\to\output
Input: dotnet publish -c Release -r win-x64 --self-contained true
Output:
PLAINTEXT
Copy
Project XYZ (.NETCoreApp,Version=v3.1) was compiled using framework .NET Core 3.1.0 (aspnetcore-runtime-3.1.2).
  Microsoft.NETCore.App 3.1.2 [C:\path\to\app\bin\Release\netcoreapp3.1\runtimes]
  Microsoft.WindowsDesktop.App 3.1.2 [C:\Program Files\dotnet\shared\Microsoft.WindowsDesktop.App]

Publishing to C:\path\to\output
```

**Data:**

```text
- `win-x64` is an example of a target runtime identifier.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Framework-DependentDeployment
6. Runtime Installed?
7. Application Runs
8. Install Runtime; ApplicationFails
9. Self-Contained Deployment
10. All Dependencies Included?
11. Application RunsIndependently
12. Package Size Larger; SlowInitial Download

