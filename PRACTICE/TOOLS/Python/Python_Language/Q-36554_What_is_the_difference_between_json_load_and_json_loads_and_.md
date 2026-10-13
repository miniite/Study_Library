---
type: "practice"
difficulty: "Basic"
---

# What is the difference between 'json.load' and 'json.loads', and 'json.dump' and 'json.dumps'?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Parsing JSON Safely](<../../../../TOOLS/Python/Python_Language/Parsing_JSON_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to distinguish four functions from Python's built-in json module: json.load, json.loads, json.dump, and json.dumps. This is a basic-concept question for backend/fullstack roles, so the interviewer is probing whether I actually understand the difference between serialization vs deserialization and, crucially, the difference between working with a file object vs an in-memory string. The common trap is that candidates memorize "s means string" without understanding what the functions consume and produce. A strong answer nails the direction (Python object ↔ JSON) and the medium (string ↔ file), and mentions the return types, which are the source of most bugs.

Plan: define serialization/deserialization, group the four functions on two axes (direction and medium), give the mnemonic for s, then show concrete usage and common mistakes.

First principles: two things vary

JSON is a text format. In Python, we constantly convert between native Python objects (dict, list, str, int, float, bool, None) and their JSON text representation. Two independent things vary across these four functions:

Direction

dump / dumps = serialize / encode: Python object → JSON text.
load / loads = deserialize / decode / parse: JSON text → Python object.

Medium (this is what the s suffix means)

The s stands for string. Functions with s (dumps, loads) work with an in-memory string.
Functions without s (dump, load) work with a file-like object (anything with .write() or .read(), e.g., an open file, io.StringIO, a socket wrapper).

So the four functions are just the 2×2 combination of {serialize, deserialize} × {string, file}.

The four functions precisely
json.dumps(obj) — takes a Python object, returns a JSON str. Nothing is written anywhere; you get the string back.
json.dump(obj, fp) — takes a Python object and a writable file object fp, and writes the JSON text directly into that file. It returns None — this is the single most common bug (x = json.dump(obj, f) leaves x as None).
json.loads(s) — takes a JSON str (or bytes in Python 3.6+), parses it, and returns the corresponding Python object.
json.load(fp) — takes a readable file object fp, reads its full contents, parses the JSON, and returns the Python object.

A neat identity to remember it all: json.dump(obj, fp) is essentially fp.write(json.dumps(obj)), and json.load(fp) is essentially json.loads(fp.read()).

When to use which
Reading/writing an actual .json file on disk → use the file versions (load/dump) with a with open(...) context manager. It streams straight to/from the file handle, so you avoid building a giant intermediate string.
Handling JSON that arrives as a string in memory — an HTTP request/response body, a Kafka/SQS message, a value from Redis, a webhook payload — → use the string versions (loads/dumps).

In backend web work this matters a lot: request.get_data() gives you a string, so you call json.loads. Building a response, most frameworks want a string, so you call json.dumps (or the framework's jsonify which wraps it).

Useful keyword arguments (shows depth)
json.dumps(obj, indent=2) → pretty-printed, human-readable output.
json.dumps(obj, sort_keys=True) → deterministic key order, handy for hashing/diffing configs.
json.dumps(obj, separators=(",", ":")) → compact output (no spaces), smaller payloads over the wire.
json.dumps(obj, default=...) and json.loads(s, object_hook=...) → custom handling of non-standard types (e.g., datetime, Decimal).
json.dumps(obj, ensure_ascii=False) → keep non-ASCII characters (like emoji or Devanagari) as-is instead of escaping them.

#### Common mistakes
to call out
Using json.dump when you meant json.dumps and then trying to send/print its None return value.
Passing a filename string to json.load instead of an open file object — json.load("data.json") fails; you need json.load(open("data.json")).
Confusing json.loads with json.load when the input is already a string (calling json.load on a string calls .read() on it and errors).
Forgetting that JSON has no native datetime, set, tuple (becomes a list), bytes, or Decimal — those need custom encoders.
Summary — what a strong answer emphasizes

load/dump handle files, loads/dumps handle strings (s = string). load* decode JSON→Python; dump* encode Python→JSON. The subtle correctness points an interviewer wants to hear: dumps/loads/load return a value, while dump returns None and writes to the file as a side effect. Mentioning the fp.write(json.dumps(...)) equivalence and a couple of real kwargs (indent, default) signals genuine day-to-day usage rather than rote memorization.

**Comparison:**

```
Function	Direction	Works with	Returns
json.dumps(obj)	Serialize (Python → JSON)	in-memory string	JSON str
json.dump(obj, fp)	Serialize (Python → JSON)	file object fp	None (writes to file)
json.loads(s)	Deserialize (JSON → Python)	in-memory string	Python object
json.load(fp)	Deserialize (JSON → Python)	file object fp	Python object
```

**Code:**

```python
import json

data = {"name": "Ada", "roles": ["admin", "dev"], "active": True, "age": None}

# --- STRING versions (the 's') ---
text = json.dumps(data)          # Python object -> JSON string  (RETURNS str)
print(text)                       # {"name": "Ada", "roles": ["admin", "dev"], ...}

back = json.loads(text)          # JSON string -> Python object   (RETURNS dict)
print(back["roles"][0])           # admin

# --- FILE versions (no 's') ---
with open("user.json", "w") as f:
    ret = json.dump(data, f)      # WRITES to file; RETURNS None
    print(ret)                    # None  <-- classic gotcha

with open("user.json") as f:
    obj = json.load(f)            # reads file -> Python object    (RETURNS dict)
    print(obj == data)            # True

# The equivalence that ties it together:
#   json.dump(obj, fp)  ==  fp.write(json.dumps(obj))
#   json.load(fp)       ==  json.loads(fp.read())

# Handy kwargs
print(json.dumps(data, indent=2, sort_keys=True))
```

**Example input / output:**

```

Input Python object:

PYTHON
Copy
{"name": "Ada", "active": True, "age": None}

json.dumps(obj) returns the string:

CODE
Copy
{"name": "Ada", "active": true, "age": null}

(Note: Python True→true, None→null.)

json.loads('{"name": "Ada", "active": true, "age": null}') returns:

PYTHON
Copy
{'name': 'Ada', 'active': True, 'age': None}

json.dump(obj, f) → writes that same JSON text into the file f, and the call itself evaluates to None.

json.load(f) → reads the file and returns the dict above.
```

**Flow:**

1. json.dumps()
2. json.loads()
3. json.dump(obj, fp)
4. json.load(fp)
5. Python object (dict/list/...)
6. JSON string
7. JSON file / stream

