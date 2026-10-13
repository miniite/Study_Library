---
type: "practice"
difficulty: "Intermediate"
---

# How does FastAPI use type hints and Pydantic to validate request bodies automatically?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/Python_Language/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks how FastAPI uses Python type hints and Pydantic to validate request bodies automatically. The interviewer wants me to explain the mechanism, not just say "it validates for you." A strong answer traces the chain: type hint on a parameter → FastAPI inspects the signature → recognizes a Pydantic model → builds a schema → parses/validates/coerces the incoming JSON → raises a structured 422 on failure or hands you a typed object on success. For an intermediate backend role I should show I understand where the magic lives (Pydantic does validation; FastAPI does the wiring and HTTP glue), and the practical implications (error format, coercion, nested models, performance).

#### Assumptions
I'd state: we're talking about a JSON request body (Pydantic model), as opposed to path/query/header params (which use type hints too but bind from different parts of the request). I'll assume Pydantic v2 (current default in modern FastAPI), noting v1 differences where they matter.

Step 1 — Type hints are runtime-introspectable metadata

Python type hints (PEP 484) aren't just for the type checker — they're stored on the function object and readable at runtime via inspect.signature() and typing.get_type_hints(). FastAPI leans entirely on this. When you write:

PYTHON
@app.post("/items")
def create(item: Item):   # Item is a Pydantic model
    ...

FastAPI inspects create's signature once, at startup, and sees a parameter item annotated with Item. This introspection is the whole trick: the annotation tells FastAPI what shape the input must be and where to bind it from.

Step 2 — FastAPI decides WHERE each parameter comes from

FastAPI applies a set of rules to each parameter based on its type and declaration:

If the annotation is a Pydantic BaseModel, FastAPI treats it as the request body (parsed from the JSON payload).
If the parameter name appears in the path (e.g. /items/{id}), it's a path param.
If it's a singular type (int, str, etc.) not in the path, it's a query param.
Special markers (Query(...), Path(...), Body(...), Header(...), Depends(...)) override these defaults.

So type hints do double duty: they define both the validation schema and the binding source. For body validation, the key branch is "annotation is a BaseModel → parse the body against it."

Step 3 — Pydantic compiles the model into a validator

A Pydantic model is a class whose fields are themselves type-annotated:

PYTHON
class Item(BaseModel):
    name: str
    price: float
    tags: list[str] = []

When this class is defined, Pydantic's metaclass reads those field annotations and compiles a validation schema for the model. In Pydantic v2, this schema is executed by a Rust core (pydantic-core), which is why validation is fast. The compiled validator knows: each field's type, whether it's required (no default) or optional (has a default / is Optional), and any constraints declared via Field(...) (e.g. gt, max_length, pattern).

FastAPI also uses this same schema to generate the OpenAPI/JSON Schema that powers the /docs UI — one declaration, three outcomes: validation, serialization, and documentation.

Step 4 — Per request: parse → validate → coerce → inject or reject

On each incoming request, FastAPI:

Reads the raw body and JSON-decodes it into a Python dict.
Calls the Pydantic model's validator on that dict (Item.model_validate(data) in v2 terms).
Pydantic then, field by field:
Checks presence of required fields (missing → error).
Checks/coerces types. Pydantic does sensible coercion: the JSON string "3.5" for a float field becomes 3.5; "true" for a bool becomes True. It rejects nonsense like "abc" for a float. (v2 is stricter than v1 about some coercions.)
Enforces constraints from Field(...) (ranges, lengths, regex).
Recursively validates nested models and containers (list[Tag], dict[str, int]).
If validation succeeds, FastAPI injects a fully typed Item instance into your handler — you never touch the raw dict, and your editor gets autocomplete.
If validation fails, Pydantic raises a ValidationError; FastAPI catches it and returns a 422 Unprocessable Entity with a structured JSON list of every error (location, message, type). Your handler code never runs on bad input — validation is a gate before business logic.

This is the core value: you declare the contract once with types; FastAPI + Pydantic enforce it automatically, uniformly, and with good error messages — no hand-written if not isinstance(...) boilerplate.

Step 5 — Response side and extra powers

The symmetry is worth mentioning. response_model=Item (or a return annotation) makes FastAPI serialize and filter the output through Pydantic too — stripping fields not in the model, coercing to JSON-safe types, and (usefully) preventing accidental leakage of internal fields. You also get:

Nested/recursive validation for free (models inside models inside lists).
Field(...) constraints and custom validators (@field_validator in v2) for domain rules (e.g., "amount must be a multiple of 100").
Annotated[...] to attach metadata cleanly, e.g. Annotated[int, Field(gt=0)].

#### Common mistakes
& follow-ups
Mistake: thinking FastAPI does the validating. It doesn't — Pydantic does. FastAPI does the introspection, binding, and HTTP glue (choosing body vs query, returning 422). Being precise about this division earns credibility.
Mistake: forgetting the 422 shape. Validation errors are 422 (not 400) by default, with a detail array. Interviewers like when you know this.
Mistake: expecting strict typing everywhere. Pydantic coerces by default ("3" → 3); if you need strict types, you use Strict/StrictInt or v2 strict mode.
Mistake: a plain (non-BaseModel) param becomes a body. Singular types default to query params; to force a scalar into the body you need Body(...).

#### Follow-ups
"How would you add a custom rule?" (@field_validator). "How do you validate query params?" (same type-hint mechanism, Query(...)). "v1 vs v2 differences?" (v2 uses pydantic-core in Rust, model_validate/model_dump renamed from parse_obj/dict, stricter coercion).
Summary

FastAPI turns type hints into an executable contract. At startup it introspects each handler's signature; a parameter typed as a Pydantic BaseModel is bound to the JSON body. Pydantic compiles that model into a fast (Rust-backed in v2) validator that, per request, parses the JSON, checks required fields, coerces types, enforces Field constraints, and recurses into nested models. Success → your handler gets a clean, typed object; failure → an automatic, structured 422 before your code runs. The same declaration also drives response serialization and OpenAPI docs — declare once, validate/serialize/document for free.

**Comparison:**

```
Concern	Who handles it	How
Read function signature / annotations	FastAPI	inspect.signature at startup
Decide body vs query vs path	FastAPI	Type is BaseModel → body; scalar → query; in path → path
Parse JSON payload	FastAPI	Decode raw body to dict
Presence / type / coercion / constraints	Pydantic	Compiled validator (model_validate), Rust core in v2
Nested & container validation	Pydantic	Recursion over model fields
Return 422 on failure	FastAPI	Catches ValidationError → structured detail
Serialize response / filter fields	FastAPI + Pydantic	response_model / return annotation
Generate OpenAPI docs	FastAPI + Pydantic	JSON Schema from the same model
```

**Code:**

```python
from typing import Annotated
from fastapi import FastAPI, Body
from pydantic import BaseModel, Field, field_validator

app = FastAPI()

class Address(BaseModel):          # nested model -> recursively validated
    city: str
    pincode: str = Field(pattern=r"^\d{6}$")

class User(BaseModel):
    name: str = Field(min_length=1, max_length=80)
    age: int = Field(ge=0, le=120)          # constraint enforced by Pydantic
    email: str
    address: Address                        # nested Pydantic model
    tags: list[str] = []                    # container, each item validated

    @field_validator("email")               # custom domain rule (Pydantic v2)
    @classmethod
    def must_have_at(cls, v: str) -> str:
        if "@" not in v:
            raise ValueError("invalid email")
        return v.lower()

@app.post("/users")
def create_user(user: User):        # <-- type hint = body + validation schema
    # 'user' is a validated, typed User instance; bad input never reaches here.
    return {"created": user.name, "email": user.email}

# A scalar in the body needs Body(...) (else it'd be a query param):
@app.post("/ping")
def ping(count: Annotated[int, Body(gt=0)]):
    return {"count": count}
```

**Example input / output:**

```text
# Valid — coercion + nested validation succeed
POST /users
{ "name": "Asha", "age": 30, "email": "ASHA@x.com",
  "address": {"city": "Pune", "pincode": "411001"}, "tags": ["a","b"] }
-> 200 OK
{ "created": "Asha", "email": "asha@x.com" }   # email lowercased by validator

# Invalid — multiple failures reported at once, handler never runs
POST /users
{ "name": "", "age": 200, "email": "nope",
  "address": {"city": "Pune", "pincode": "41"} }
-> 422 Unprocessable Entity
{
  "detail": [
    {"loc": ["body","name"],            "msg": "String should have at least 1 character"},
    {"loc": ["body","age"],             "msg": "Input should be less than or equal to 120"},
    {"loc": ["body","email"],           "msg": "Value error, invalid email"},
    {"loc": ["body","address","pincode"],"msg": "String should match pattern '^\\d{6}$'"}
  ]
}
```

**Data:**

```

JSON Schema that Pydantic derives for User (feeds OpenAPI /docs):

JSON
Copy
{
  "title": "User",
  "type": "object",
  "required": ["name", "age", "email", "address"],
  "properties": {
    "name":  {"type": "string", "minLength": 1, "maxLength": 80},
    "age":   {"type": "integer", "minimum": 0, "maximum": 120},
    "email": {"type": "string"},
    "address": {"$ref": "#/$defs/Address"},
    "tags":  {"type": "array", "items": {"type": "string"}, "default": []}
  }
}
```

**Flow:**

1. Your handler
2. Pydantic (validator)
3. FastAPI
4. Client
5. Your handler
6. Pydantic (validator)
7. FastAPI
8. Client
9. at startup: introspect signature,
10. param typed as BaseModel => body
11. alt
12. [valid]
13. [invalid]
14. POST /users {JSON body}
15. JSON-decode body to dict
16. model_validate(dict)
17. typed User instance
18. inject User
19. 200 + result
20. ValidationError (all field errors)
21. 422 + detail[] (handler never runs)

