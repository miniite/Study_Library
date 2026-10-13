---
type: "practice"
difficulty: "Intermediate"
---

# What is a webhook in the context of GitHub, and how would you use one to trigger an external Jenkins job on push?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reducing Actions Costs](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reducing_Actions_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A webhook in GitHub is essentially a callback mechanism that allows GitHub to notify other services (like Jenkins) when specific events occur, such as code being pushed or merged. In this scenario, we can set up a webhook to trigger an external Jenkins job whenever a push event happens on the repository.

A webhook in GitHub enables real-time communication between different services, making it possible to automate workflows and integrate various tools within your development process. For example, you could configure a webhook to run automated tests or deploy code after receiving a commit push. In this answer, we'll discuss how to set up such a webhook using Jenkins.

Restating the Question: The question is asking about understanding what webhooks are in the context of GitHub and how to use them to trigger an external Jenkins job upon a push event. We need to assume that both GitHub and Jenkins are already set up and running. Let's start by explaining webhooks from first principles before diving into the setup process.

Understanding Webhooks: A webhook is essentially a URL that you provide to the service provider (in this case, GitHub), which acts as an endpoint for receiving HTTP POST requests when specific events occur. When these events happen, GitHub sends a POST request to your provided webhook URL with details about the event.

Setting Up Jenkins: First, ensure Jenkins is properly configured and accessible over the network. You should have Jenkins set up with plugins like "GitHub Plugin" or similar ones that allow interaction with GitHub repositories.

Creating a Webhook in GitHub:

Log into your GitHub account.
Navigate to the repository where you want to trigger the Jenkins job upon push.
Go to Settings > Webhooks and click on Add webhook.
In the "Payload URL" field, enter the appropriate URL for your Jenkins server. This URL should point to a script or endpoint in Jenkins that can process incoming webhook requests.
Set up the "Content type" as application/json.
Choose which events you want to trigger the webhook (e.g., push events).
Click on Add webhook.

Jenkins Scripting: On your Jenkins server, create a script or endpoint that listens for incoming POST requests from GitHub. This script should parse the JSON payload received in the request body and determine if it's a push event. If so, it can trigger the desired job.

#### Example
Jenkins Job Configuration:

Go to Jenkins > Manage Jenkins > Configure System.
Under "Global Properties", add an Environment Variable named WEBHOOK_URL with your GitHub webhook URL.
Create a new Freestyle project and configure it as needed, then under Build Triggers, enable the option for GitHub hook trigger for GITScm polling.

Testing: After setting up, push some code to the repository and ensure that Jenkins picks up the change via the webhook.

#### Closing Summary
+ Common Mistakes/Follow-Ups: Setting up a webhook in GitHub to trigger an external Jenkins job is crucial for automated CI/CD pipelines. Key points include ensuring proper URL configuration, event selection, and robust scripting on the Jenkins side. Common mistakes might include incorrect URL configurations or not properly handling different events like branches or tags.

For follow-ups:

How would you handle cases where multiple repositories need similar setup?
What other services could benefit from such a webhook integration?

**Code:**

```bash
# Example Jenkins Script for Handling Webhooks
if [[ $WEBHOOK_EVENT == "push" ]]; then
    echo "Push event received, triggering build..."
    # Trigger the Jenkins job here.
fi
```

**Flow:**

1. GitHub Push Event
2. Send POST Request to JenkinsWebhook URL
3. Jenkins Script ReceivesRequest
4. Parse JSON Payload & CheckEvent Type
5. [Trigger Jenkins Job]

