---
type: "practice"
difficulty: "Basic"
---

# What does 'if __name__ == \\"__main__\"\":' do and why is it used?"

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Subprocess Safety](<../../../../TOOLS/Python/Python_Language/Subprocess_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Explain the meaning of the line if __name__ == "__main__":, what the __name__ variable is, and why Python programmers put this guard at the bottom of their scripts.

What the interviewer is probing: Whether you understand how Python modules are imported vs. executed, that Python has no special main() function like C/Java, and that you know the practical reason for the idiom (making a file usable both as a runnable script and as an importable library without side effects). It is a "basic concept" question, so they want a clear, correct, jargon-free explanation with a small example — not a lecture on internals.

#### Assumptions
/ clarifications: None really needed. I'll assume standard CPython and a normal .py file. I'll explain (1) what __name__ is, (2) what value it takes in the two scenarios, (3) why the guard matters, and (4) common uses and pitfalls.

From first principles
1. Every Python module has a __name__

When Python loads any .py file, it automatically creates a special built-in variable inside that file called __name__. Its value depends on how the file was loaded:

If you run the file directly — e.g. python app.py — Python sets __name__ to the string "__main__". This file is the "entry point" of the program.
If the file is imported by another module — e.g. import app — Python sets __name__ to the module's own name, "app" (the filename without .py).

So __name__ answers the question: "Am I the file that was launched, or am I being imported by someone else?"

2. What the line does
PYTHON
if __name__ == "__main__":
    main()   # or whatever "run this now" code you want

This is just a normal if statement. It reads: "If this file is the one being run directly, then execute the block below. If this file was merely imported, skip the block."

There is nothing magical about the words — __name__ is an ordinary variable and "__main__" is an ordinary string. Python does not have a required main() function that runs automatically (unlike C or Java). Instead, when you run a file, all top-level code executes top to bottom. This idiom lets you choose which code runs only on direct execution.

3. Why it is used

The core reason is: a Python file is often two things at once — a runnable program AND a reusable library. Consider a file calc.py:

PYTHON
def add(a, b):
    return a + b

print(add(2, 3))   # side effect!

If another file does import calc just to reuse add(), that print(add(2, 3)) would fire during import, producing unwanted output (a side effect). Worse, if the top-level code started a web server, opened a file, or ran a long computation, importing the module would trigger all of that.

Wrapping the "run now" code in the guard fixes this:

PYTHON
def add(a, b):
    return a + b

if __name__ == "__main__":
    print(add(2, 3))   # runs only when you do `python calc.py`

Now import calc gives you the add function with no side effects, while python calc.py still runs the demo/CLI. This is why you see the guard in virtually every well-written script.

4. Common, concrete uses
CLI entry point / demo code: put argument parsing and the program's actual work behind the guard.
Quick tests / self-checks: run some assertions when the file is executed directly, without affecting importers.
Keeping libraries clean: ensures your module can be imported for its functions/classes without executing anything.
multiprocessing on Windows/macOS: the guard is required there — child processes re-import the main module, and without the guard you get an infinite spawn loop / runtime error.
5. Best-practice pattern

Keep the guard tiny — just call a function:

PYTHON
def main():
    ...  # real logic here

if __name__ == "__main__":
    main()

Putting logic inside main() (rather than sprawling under the if) keeps names out of the module's global namespace and makes the code testable — a test can import the module and call main() or the smaller functions directly.

Summary — what a strong answer emphasizes
__name__ is a per-module variable Python sets automatically: "__main__" when the file is run directly, or the module name when imported.
The if __name__ == "__main__": block therefore runs only on direct execution, not on import.
Purpose: let one file be both an importable library (no side effects) and a runnable script/CLI.
Bonus points: mention it's required for multiprocessing, and that best practice is to delegate to a main() function.

#### Common mistakes
to avoid: thinking Python auto-runs a main() function; writing "__main__" as "main" (it needs the double underscores); or believing the guard is mandatory (it is not — a file with no import-time side effects works fine without it, but it's a widely followed convention).

Likely follow-up probes: "What is __name__ when imported?" (the module's name). "Is main() special in Python?" (no — only the guard makes execution conditional). "Why does multiprocessing need it?"

**Comparison:**

```
How the file is loaded	Value of __name__	Does the guard block run?
Run directly: python calc.py	"__main__"	Yes
Imported: import calc	"calc" (module name)	No
Run as module: python -m calc	"__main__"	Yes
```

**Code:**

```
calc.py — usable as BOTH a library and a script

def add(a, b): return a + b

def main(): print("2 + 3 =", add(2, 3))

runs only when executed directly: python calc.py
skipped when another file does: import calc

if name == "main": main()

CODE
Copy

```python
# other.py — importing calc does NOT trigger main()
import calc
print(calc.add(10, 20))   # -> 30, and NO "2 + 3 = 5" line printed
```

**Example input / output:**

```

Given the files above:

TEXT
Copy
$ python calc.py
2 + 3 = 5

$ python other.py
30

Note: running other.py (which imports calc) prints only 30. The main() in calc.py did NOT fire, because on import calc.__name__ == "calc", not "__main__".

Quick demonstration of the variable itself:

TEXT
Copy
# inside calc.py, run directly:
print(__name__)   # -> __main__

# inside calc.py, when imported by other.py:
print(__name__)   # -> calc
```

**Flow:**

1. Run directly / python -m
2. Imported by anothermodule
3. True
4. False
5. Python loads a .py file
6. How was it loaded?
7. __name__ = '__main__'
8. __name__ = module name
9. if __name__ == '__main__'
10. Run the guarded block e.g.main
11. Skip block; only defs/classesavailable

