---
type: "practice"
difficulty: "Intermediate"
---

# How do you add a trailing-slash route in Flask and what redirect behavior does Flask apply when the slash is missing?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Routing and Blueprints](<../../../../TOOLS/Python/Flask/Routing_and_Blueprints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to handle trailing-slash routes in Flask, we'll first summarize the key points, explain the default behavior, discuss potential issues, and explore a custom solution. We'll then walk through implementing these concepts step-by-step.

In Flask, a request without a trailing slash will not match a route ending with /. By default, Flask does not automatically redirect to the URL with a trailing slash unless you explicitly configure it. This can lead to inconsistencies in your application's URLs and potentially affect user experience or SEO if the same content is accessed via different URLs.

#### Triage
What the question is asking: How to handle requests that don't include a trailing slash in Flask routes.
What the interviewer might be probing: Your understanding of URL routing, request handling, and redirect mechanisms in Flask. Additionally, your ability to implement custom behavior based on application requirements.

#### Assumptions
We assume you are working with a typical web application using Flask as the backend.

#### Solution

To handle trailing-slash routes in Flask, we need to explicitly configure our application to automatically add or redirect from URLs without a trailing slash. This can be achieved by modifying the routing logic and implementing custom error handlers.

Step-by-Step Implementation
Define Routes: First, define your main route with a trailing slash.
Create Custom Error Handler: Implement an error handler for 404 to redirect to the URL with a trailing slash if necessary.
Check for Trailing Slash in URL: Use a utility function or regular expression to check if a URL is missing a trailing slash.

Let's walk through these steps with code examples and explanations:

Code Implementation
PYTHON
from flask import Flask, request, redirect, url_for

app = Flask(__name__)

# Step 1: Define Routes
@app.route('/users/<int:user_id>/', methods=['GET'])
def user_profile(user_id):
    # Your logic here to fetch the profile of a specific user
    return f"User Profile for ID {user_id}"

# Step 2: Create Custom Error Handler
@app.errorhandler(404)
def handle_404(e):
    # Check if the request is missing a trailing slash and redirect accordingly
    if not request.url.endswith('/'):
        new_url = url_for(request.endpoint, **request.view_args) + '/'
        return redirect(new_url, code=302)
    else:
        # Return the default 404 error page or custom content here
        return "Page Not Found", 404

# Run the Flask application
if __name__ == '__main__':
    app.run(debug=True)
Explanation
Route Definition: The route /users/<int:user_id>/ is defined, which includes a trailing slash.
Custom Error Handler: An error handler for 404 is created to check if the URL does not end with a trailing slash. If it doesn't, the function constructs a new URL with the trailing slash and performs a redirect.
Run Application: The application is run in debug mode to facilitate testing.

#### Closing Summary

By implementing custom error handling and route definitions in Flask, you can ensure that URLs without trailing slashes are redirected to their correct counterparts, improving consistency and user experience. This approach requires careful consideration of edge cases and the overall impact on your application's architecture.

#### Common Mistakes
and Follow-Ups
Forgetting to include the trailing slash: Always ensure routes are defined with trailing slashes.
Incorrect redirect logic: Double-check that your custom error handler correctly identifies missing slashes and applies the appropriate redirects.
Performance implications: Be mindful of the additional overhead introduced by redirects, especially in high-traffic applications.

