---
type: "practice"
difficulty: "Deep"
---

# How do you mount a sub-application or another ASGI app such as a GraphQL server inside a FastAPI app?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Service Architecture and Versioning](<../../../../TOOLS/Python/FastAPI/Service_Architecture_and_Versioning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate a sub-application like a GraphQL server into a FastAPI application, the key concept is "mounting" this sub-app as a sub-path within the main FastAPI app. This allows you to route requests from your main API to the separate GraphQL service based on URL paths.

First, let’s clarify what the question is probing: the interviewer wants to ensure you understand how to handle multiple ASGI applications in a single FastAPI instance and possibly explore performance trade-offs or potential architectural implications.

#### Assumptions

You have basic knowledge of FastAPI.
The sub-application (e.g., GraphQL server) works as an ASGI application.
The main FastAPI app is also configured as an ASGI app for seamless integration.

To mount a sub-app in FastAPI, follow these steps:

Install the Required Packages: Ensure you have fastapi and any other required dependencies installed via pip.

Create the Sub-Application: Define your GraphQL server or another application that works with ASGI.

Mount the Sub-Application: Use the app.mount() method to attach the sub-application as a path within the main FastAPI app.

Here’s an example:

PYTHON
from fastapi import FastAPI

# Step 1: Install dependencies and create your sub-app
def graphql_app():
    from starlette.applications import Starlette
    from starlette.routing import Route, Mount
    from starlette.graphql import GraphQLApp
    from graphene.schema import Schema
    # Import or define your schema here...

    return Starlette(
        debug=True,
        routes=[
            Route("/graphql", GraphQLApp(schema=Schema())),
        ]
    )

# Step 2: Create the main FastAPI app and mount the sub-app
main_app = FastAPI()

@main_app.on_event("startup")
async def startup():
    # Mount the sub-application as '/graphql'
    main_app.mount("/graphql", graphql_app())

# Run your application (for demonstration purposes)
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(main_app, host="0.0.0.0", port=8000)

In this setup:

graphql_app is a function that returns an ASGI application.
The mount method of the main FastAPI app is used to attach the sub-app at /graphql.
Requests sent to /graphql will be handled by the mounted GraphQL sub-application.

This approach ensures clear separation of concerns and allows you to manage multiple services within a single FastAPI instance, enhancing modularity and maintainability.

#### Closing Summary
Mounting a sub-application in FastAPI involves creating an ASGI sub-app and using app.mount() to attach it. This method is useful for integrating specialized services like GraphQL or other APIs into your main application.

#### Common Mistakes/Follow-ups

Not ensuring the sub-app is properly configured as an ASGI app.
Forgetting to handle CORS or authentication if these are part of the sub-app.
Ignoring potential performance implications when multiple ASGI apps are running in a single process.

