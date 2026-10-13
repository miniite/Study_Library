---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between subprocess.run and os.system, and why is subprocess preferred in automation scripts?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Subprocess Safety](<../../../../TOOLS/Python/Python_Language/Subprocess_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between subprocess.run and os.system in Python, we need to start by grasping their fundamental purposes and how they operate. Essentially, both are used to execute shell commands from within a Python script. However, there are significant differences that make subprocess.run more versatile and preferred for automation scripts.

Firstly, let's look at what each does:

os.system: This function executes a command via the user’s default shell (e.g., bash on Unix-like systems). It returns an integer status code indicating whether the command was executed successfully. However, it has several limitations and drawbacks.

subprocess.run: This is part of Python's subprocess module and provides more control over the process execution. It can run commands in a subshell and interact with them in various ways (like capturing output or setting timeout limits).

Now, let’s break down why subprocess.run is generally preferred:

Control Over Process Execution:

With os.system, you can't easily capture the standard input/output/error streams of the command being executed.
subprocess.run, on the other hand, allows you to specify stdin, stdout, and stderr directly via arguments like stdout=subprocess.PIPE or stderr=subprocess.PIPE.

Error Handling:

In os.system, if a command fails (e.g., non-zero return code), it's harder to handle errors compared to subprocess.run.
With subprocess.run, you can check the returncode, stdout, and stderr directly.

Flexibility in Execution:

os.system always runs commands through the shell, which means it has all the nuances of shell command execution.
subprocess.run is more flexible; it allows running raw commands or lists of arguments without going through a shell, which can be safer.

Security and Reliability:

Using os.system poses security risks due to its reliance on the shell. Malicious input in the command string could lead to injection attacks.
subprocess.run is generally more secure because it doesn’t use the shell by default, making it harder for malicious code to affect your script.

Error Handling and Debugging:

The output from os.system can be difficult to parse and debug if an error occurs.
With subprocess.run, you get detailed information about what happened (returncode, stdout, stderr), making it easier to diagnose issues.

#### Closing Summary
In summary, while both functions serve the purpose of executing shell commands in Python, subprocess.run is preferred due to its enhanced control and flexibility. It offers better error handling, security, and debugging capabilities, which are crucial for robust automation scripts.

#### Common Mistakes
and Follow-Ups:

Mistake: Using os.system when you need finer-grained control over the command execution.

#### Follow-Up
Discuss scenarios where shell commands might still be necessary (e.g., complex file operations) but always consider the risks associated with it.

