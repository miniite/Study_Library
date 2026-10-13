---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between a module and a package in Python and how imports resolve them.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Functions Modules and Reuse](<../../../../TOOLS/Python/Python_Language/Functions_Modules_and_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Two things — (1) the definitions: what is a module vs a package in Python? (2) the mechanics: when you write import x or from x import y, how does Python find and load them?

What the interviewer is probing: This is a "basic concept" question, so they want clear, correct fundamentals — no hand-waving. Can you define the terms precisely, name the marker of a package (__init__.py), and describe the import-resolution steps (sys.modules cache → sys.path search → the module cache)? A confident, well-organized answer here signals you actually understand how a Python project is laid out.

#### Assumptions
Python 3. I'll define both terms, distinguish regular vs namespace packages briefly, then walk through import resolution step by step with a concrete example.

Plan: module → package → the __init__.py marker → import resolution algorithm → a worked example → summary and gotchas.

What is a module?

A module is a single .py file. Its name is the file name without the extension. When you import math or import myutils, Python runs that file once and gives you a module object whose attributes are the names defined at the top level of the file (functions, classes, variables).

CODE
myutils.py   -> module "myutils"

Everything defined in myutils.py becomes myutils.something after import. A module is Python's basic unit of code organization and namespacing.

What is a package?

A package is a directory of modules — a way to group related modules under a dotted namespace like myapp.services.payment. The classic ("regular") package is a directory that contains an __init__.py file:

CODE
myapp/                 <- package
├── __init__.py        <- marks myapp as a package; runs on first import of myapp
├── config.py          <- module myapp.config
└── services/          <- sub-package
    ├── __init__.py
    └── payment.py     <- module myapp.services.payment

__init__.py can be empty (its presence alone marks the directory as a package) or it can contain package-level setup / re-exports. Packages can nest arbitrarily, giving you dotted paths (myapp.services.payment).

One nuance worth mentioning: since Python 3.3 there are also namespace packages — a directory without __init__.py can still act as a package, and one logical package can be split across multiple directories on sys.path. But the default, explicit choice for a real project is a regular package with __init__.py. Simple rule to remember: module = one .py file; package = a folder of modules (marked by __init__.py).

How imports resolve — the algorithm

When you execute import spam, Python does roughly this:

Check the cache first. Look in sys.modules, a dict of already-imported modules. If spam is there, use it as-is. This is why a module's top-level code runs only once per process no matter how many times it's imported.
Find the module using the finders on sys.meta_path. The key one is the path-based finder, which searches each directory in sys.path in order. sys.path typically contains: the directory of the entry script (or '' = cwd for the REPL), the PYTHONPATH env var entries, and the standard-library / site-packages directories.
Load and execute. The finder returns a loader; Python creates a new module object, inserts it into sys.modules before running the file (this is how circular imports can partially work), then executes the file's top-level code to populate it.
Bind the name. import spam binds the name spam in your current namespace to that module object.

For a package import like import myapp.services.payment, Python resolves it left-to-right: import myapp (running myapp/__init__.py), then myapp.services (running its __init__.py), then the payment module — caching each in sys.modules along the way.

from x import y does the same find + load of module x, then binds the attribute y (which may be a name inside module x, or a sub-module) into your namespace — not the whole module.

Absolute vs relative imports: absolute imports name the full path from a top-level package (from myapp.services import payment) and are the recommended default. Relative imports use dots to mean "relative to the current package" (from . import payment, from ..config import SETTINGS) and only work inside a package.

Summary — what a strong answer emphasizes

A module is a single .py file; a package is a directory of modules that (in the regular form) contains __init__.py and enables dotted namespaces like pkg.sub.mod. Imports resolve by: check sys.modules cache → search sys.path via finders → create, cache, and execute the module once → bind the name. The sys.modules cache is why top-level code runs only once; sys.path is the ordered search list; __init__.py is the package marker (and runs on first import).

#### Common mistakes
/ gotchas to mention: confusing a module (file) with a package (folder); forgetting __init__.py for a regular package; a local file shadowing a stdlib module (naming your file random.py next to your script breaks import random); assuming re-imports re-run the file (they don't — the cache serves them); and mixing up relative vs absolute imports. A good follow-up answer: to force a re-run you'd use importlib.reload(module).

**Comparison:**

```
Aspect	Module	Package
What it is	A single .py file	A directory of modules
Marker	(none — any .py)	__init__.py (regular package)
Namespace	module.name	dotted: pkg.sub.mod
Import example	import myutils	import myapp.services.payment
Can contain sub-items	functions/classes/vars	modules and sub-packages
```

**Code:**

```python
# --- module: myutils.py ---
PI = 3.14159
def area(r):
    return PI * r * r

# --- using the module ---
import myutils              # runs myutils.py once, creates module object
print(myutils.area(2))     # 12.566...

from myutils import area    # binds just the name `area`
print(area(2))

# --- package layout ---
# myapp/__init__.py
# myapp/config.py           -> myapp.config
# myapp/services/__init__.py
# myapp/services/payment.py -> myapp.services.payment
from myapp.services import payment      # absolute import (recommended)

# inside myapp/services/payment.py you could instead write a relative import:
# from ..config import SETTINGS

# inspect resolution machinery
import sys
print(sys.path)                         # ordered search directories
print('myutils' in sys.modules)         # True after first import (cached)
```

**Example input / output:**

```text
>>> import myutils          # 1st time: not in sys.modules -> search sys.path
                            #   -> find ./myutils.py -> execute top-level once
                            #   -> insert into sys.modules -> bind name `myutils`
>>> import myutils          # 2nd time: FOUND in sys.modules -> reused, file NOT re-run
>>> import myapp.services.payment
                            # resolves left->right:
                            #   myapp (__init__.py runs)
                            #   -> myapp.services (__init__.py runs)
                            #   -> myapp.services.payment
```

**Data:**

```text
Project tree:
project/
├── main.py                 # entry script -> its dir goes on sys.path
├── myutils.py              # a MODULE  -> import myutils
└── myapp/                  # a PACKAGE -> import myapp...
    ├── __init__.py         #   package marker (runs on first import)
    ├── config.py           #   module myapp.config
    └── services/           #   sub-PACKAGE
        ├── __init__.py
        └── payment.py      #   module myapp.services.payment
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. import spam
6. In sys.modules cache?
7. Reuse cached modulefile NOT re-run
8. Finders search sys.path inorder
9. Found?
10. ModuleNotFoundError
11. Create module object
12. Insert into sys.modules first
13. Execute file top-level codeonce
14. Bind name in caller'snamespace

