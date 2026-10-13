---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the .NET SDK and the .NET runtime, and which do you need to just run an app?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ JIT vs AOT Compilation](<../../../../TOOLS/DotNet/DotNet_Platform/JIT_vs_AOT_Compilation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with .NET applications, it's essential to understand the difference between the .NET SDK and the .NET runtime. The .NET SDK includes tools and libraries necessary for building .NET applications, while the .NET runtime is required for running those applications on a machine where the code will be executed.

To run an application, you only need the .NET runtime if your goal is to deploy and execute pre-built binaries. However, to develop, test, and build new applications, you'll need the full .NET SDK, which provides additional tools like dotnet, compilers, and development libraries.

Let's break this down step by step:

Understanding the .NET Runtime: The runtime is a component that interprets your application's code and executes it on the machine. It handles low-level tasks such as memory management, thread scheduling, and garbage collection.
Understanding the .NET SDK: The SDK includes everything you need to develop applications, including:
dotnet CLI: A command-line interface for building, running, and publishing .NET apps.
Compilers (like C#): Tools that convert source code into executable code.
Development libraries: Frameworks and libraries used in application development.

#### Triage

The question is essentially asking you to explain the difference between these two components and determine which one is necessary for running an app. The interviewer may be probing your understanding of .NET's architecture and how it supports both development and execution.

#### First-Principles Solution
Runtime Only: If someone has already built a .NET application (e.g., compiled to an executable), they need only the runtime to run this application on their machine.
SDK for Development: For developers, the SDK is essential because it includes tools and libraries that allow them to build and test applications.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the .NET runtime is what you need to execute your compiled .NET app, while the SDK is necessary if you're planning to develop new apps. A common mistake is assuming that having the runtime alone suffices for both development and execution needs. Always ensure you have the correct tools based on your intended use.

#### Follow-up
questions might include:

"What are some scenarios where you would need only the runtime?"
"How do .NET Core, .NET 5+, and other versions fit into this picture?"

**Code:**

```csharp
// Example of using dotnet CLI to run an app (assuming SDK is installed)
dotnet run --project MyProject/MyProject.csproj

// Example of running a pre-built executable with only the runtime
./MyApp.exe
```

