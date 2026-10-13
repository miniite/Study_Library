---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up a FastAPI service so it can run predictions locally during development but load a real model in production?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Packaging Models in Containers](<../../../../TOOLS/Python/FastAPI/Packaging_Models_in_Containers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up a FastAPI service that runs predictions locally during development and loads a real model in production, you need to manage your environment configuration effectively. This involves creating separate configurations for development and production, dynamically loading models based on the environment. Here's how you can approach this:

#### Overview
The core idea is to use environment-specific settings and dynamic imports to load different implementations of your model service based on whether it’s running in a local or production environment.

#### Triage
The question is probing on understanding how to manage environment-specific configurations, dynamic code loading, and the ability to switch between development and production setups without code changes.

#### First-Principles Solution
Environment Configuration: Use Python’s built-in os module or a third-party library like pydantic for environment variable handling.
Model Loading Strategy: Implement a strategy pattern where you can load different implementations based on the environment.
Dynamic Imports: Use Python's importlib to dynamically import modules based on configuration.

Here’s a step-by-step breakdown:

Define Environment Variables:

BASH
# Local (Development)
export ENV=development

# Production
export ENV=production

Create a Configuration Class:

PYTHON
from pydantic import BaseSettings

class Settings(BaseSettings):
    model_path: str = 'path/to/dev_model.pkl'

    @property
    def model(self) -> AnyModel:
        if self.model_path.startswith('path/to/'):
            return self.load_local_model()
        else:
            return self.load_production_model()

    def load_local_model(self) -> AnyModel:
        # Load the local model for development
        from local_model import LocalModel
        return LocalModel(path=self.model_path)

    def load_production_model(self) -> AnyModel:
        # Load the real production model
        from prod_model import ProdModel
        return ProdModel(path=self.model_path)

Dynamic Import of Models:

PYTHON
import importlib

class ModelService:
    def __init__(self):
        self.settings = Settings()
        if 'ENV' in os.environ and os.environ['ENV'] == 'production':
            model_module = "prod_model"
        else:
            model_module = "local_model"
        self.model_class = getattr(importlib.import_module(model_module), 'ModelClass')

Run Locally vs Production:

Local: Run the service with uvicorn app:app --reload to enable hot-reloading.
Production: Use a production-ready WSGI server like Gunicorn, e.g., gunicorn app:app.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Ensure that your model paths are correctly set in both environments.
Handle edge cases where the environment variable might not be set or incorrectly configured.
Validate the models before and after loading to ensure they are as expected.

This approach ensures flexibility and maintainability, making it easy to transition between local development and production environments without altering core logic. Common mistakes include incorrect path handling and not properly setting up environment-specific configurations.

