---
type: "practice"
difficulty: "Deep"
---

# How do you validate a FastAPI inference contract against consumers so a change in the response_model (like renaming a score field) is caught before it breaks a downstream ML pipeline?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monitoring Model Quality in APIs](<../../../../TOOLS/Python/FastAPI/Monitoring_Model_Quality_in_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that changes to an ML model's inference response don't break downstream pipelines, we need a robust validation mechanism for FastAPI response models. This involves defining and enforcing contracts between our API and its consumers. Specifically, renaming or altering fields in the response model should not result in issues with existing clients.

The key idea is to implement semantic versioning and use an intermediary layer to manage contract changes without affecting consumers immediately. Let's break this down from first principles.

First, we need to understand that FastAPI uses Python type annotations for defining response_model. These models can change over time as the model evolves, but we want to ensure compatibility with existing clients. Semantic versioning will help us manage these changes gracefully.

#### Triage

The question is asking how to validate a FastAPI inference contract against consumers so that renaming or altering fields in the response model does not break downstream ML pipelines. This involves understanding the current state of the API and defining mechanisms for backward compatibility.

#### Assumptions

The API uses semantic versioning (SemVer).
We have existing clients relying on the API.
Changes to the response_model are intentional updates.

#### Solution

Versioned Response Models: Define different versions of your response model based on SemVer principles. For example, if you're using a version like 0.2.3 for the current model structure, any changes will result in a new version (e.g., 0.3.0).

PYTHON
from fastapi import FastAPI

app = FastAPI()

# Current response model (v0.2.3)
class OldModel(BaseModel):
    score: float
    prediction: str

@app.get("/predict", response_model=OldModel)
async def predict(input_data: Dict[str, Any]):
    result = {"score": 0.95, "prediction": "positive"}
    return result

Metadata for Response Models: Introduce metadata in the response models to indicate their version and other relevant information.

PYTHON
from fastapi import FastAPI

class VersionedModel(BaseModel):
    __version__: str = "0.3.0"
    score: float
    prediction: str

@app.get("/predict", response_model=VersionedModel)
async def predict(input_data: Dict[str, Any]):
    result = {"score": 0.95, "prediction": "positive"}
    return VersionedModel(**result)

Intermediary Validation Layer: Implement a middleware or custom validation layer that checks the response version against expected versions before serving it to consumers.

PYTHON
def validate_response(response: Dict[str, Any]):
    # Assume we have a list of supported versions
    supported_versions = ["0.2.3", "0.3.0"]

    if response.get("__version__") not in supported_versions:
        raise HTTPException(status_code=415, detail="Unsupported model version")

@app.middleware("http")
async def validate_model_version(request: Request, call_next):
    response = await call_next(request)
    validate_response(response.json())
    return response

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing these steps, we ensure that any changes to the response_model are handled with care and do not break existing consumers. This approach leverages semantic versioning and adds a middleware layer for validation.

#### Common mistakes
include:

Overlooking the need for metadata in response models.
Failing to implement proper version checks and handling.
Not considering edge cases, such as deprecated versions or unexpected versions.

For further discussion, we could explore integrating these changes with a CI/CD pipeline to automate testing and deployment of new API versions. Additionally, using external tools like OpenAPI/Swagger can provide better documentation and validation for APIs.

