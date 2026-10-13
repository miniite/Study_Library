---
type: "practice"
difficulty: "Deep"
---

# How would you pin and audit NuGet packages for known vulnerabilities in a CI pipeline?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Thread Safety and Locking](<../../../../TOOLS/DotNet/DotNet_Platform/Thread_Safety_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your .NET projects are secure, it's crucial to regularly check for known vulnerabilities in the NuGet packages used. Pinning and auditing these packages can be effectively managed through a CI/CD pipeline. This process involves several steps, from initial setup to ongoing monitoring.

The core of this solution is to integrate a static analysis tool that checks package versions against a vulnerability database, then pin those versions in your project files if necessary. This ensures that you always use the latest secure versions of packages while maintaining compatibility with your codebase.

**Code:**

```csharp
// Example of a NuGet Package Management Script
using System;
using System.IO;
using NuGet.Configuration;

class Program
{
    static void Main()
    {
        string[] projects = Directory.GetFiles(".", "*.csproj", SearchOption.AllDirectories);

        foreach (string projectFile in projects)
        {
            AuditNugetPackages(projectFile);
        }
    }

    static void AuditNugetPackages(string projectFilePath)
    {
        NuGet.Configuration.Settings settings = new NuGet.Configuration.Settings();
        PackageSourceResolver resolver = new PackageSourceResolver(settings);

        var packageIds = GetPackageIds(projectFilePath, resolver);

        foreach (var id in packageIds)
        {
            string version = GetLatestSecureVersion(id, resolver);

            if (!string.IsNullOrEmpty(version))
            {
                PinNugetPackage(version, projectFilePath);
                AuditVulnerability(packageIds[id], version);
            }
        }
    }

    static string[] GetPackageIds(string projectFile, PackageSourceResolver resolver)
    {
        // Read the .csproj file and extract package IDs
        return new string[] { "Microsoft.AspNetCore.Mvc", "FluentValidation" };
    }

    static string GetLatestSecureVersion(string packageName, PackageSourceResolver resolver)
    {
        // Query a vulnerability database to get the latest secure version
        return "2.1.0";  // Hypothetical response from the database
    }

    static void PinNugetPackage(string version, string projectFile)
    {
        // Update the .csproj file with the specified package version
        File.WriteAllText(projectFile, File.ReadAllText(projectFile).Replace("Version=\"\"", $"Version=\"{version}\""));
    }

    static void AuditVulnerability(string packageName, string version)
    {
        // Log and report any vulnerabilities found for the package version
        Console.WriteLine($"Auditing {packageName} version {version}");
    }
}
```

**Example input / output:**

```text
Input:
- .csproj files in the current directory and subdirectories

Output:
- Updated .csproj files with latest secure versions of packages
- Audit logs for each package version, indicating any vulnerabilities found
```

**Data:**

```json
{
  "vulnerabilities": [
    {
      "packageId": "Microsoft.AspNetCore.Mvc",
      "versions": [
        {"version": "2.0.0", "issuesCount": 1},
        {"version": "2.1.0", "issuesCount": 0}
      ]
    },
    {
      "packageId": "FluentValidation",
      "versions": [
        {"version": "8.3.1", "issuesCount": 2},
        {"version": "9.0.0", "issuesCount": 0}
      ]
    }
  ]
}
```

