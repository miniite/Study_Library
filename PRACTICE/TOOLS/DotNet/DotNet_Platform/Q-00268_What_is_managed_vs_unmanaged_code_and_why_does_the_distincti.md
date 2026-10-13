---
type: "practice"
difficulty: "Basic"
---

# What is managed vs unmanaged code, and why does the distinction matter?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ JIT vs AOT Compilation](<../../../../TOOLS/DotNet/DotNet_Platform/JIT_vs_AOT_Compilation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a .NET environment, the key difference between managed and unmanaged code lies in how they are handled by the runtime. Managed code executes within the Common Language Runtime (CLR) on the .NET Framework or .NET Core/5+ environments, while unmanaged code runs outside of this runtime. This distinction is crucial because it impacts memory management, performance optimizations, and overall application architecture.

**Code:**

```csharp
// Example of managed code in C#
public void ManagedMethod()
{
    int a = 5;
    Console.WriteLine(a);
}

// Example of unmanaged code using System.Runtime.InteropServices
[DllImport("some.dll")]
private static extern void UnmanagedFunction();
```

