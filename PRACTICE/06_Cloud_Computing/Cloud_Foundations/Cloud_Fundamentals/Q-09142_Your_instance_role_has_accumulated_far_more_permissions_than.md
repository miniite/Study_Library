---
type: "practice"
difficulty: "Deep"
---

# Your instance role has accumulated far more permissions than the app uses and security wants it tightened. How would you discover the minimal permission set without breaking production?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Least Privilege IAM](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Least_Privilege_IAM.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this issue, we need to find a balance between maintaining application functionality and adhering to strict security policies. Our goal is to identify the minimum necessary permissions while ensuring that no production function breaks due to reduced permissions. This involves a systematic approach combining automation with manual checks.

First, let's summarize what we’re dealing with: an overly permissive instance role with many permissions assigned but not all of them being utilized by the application. Security now wants to tighten this up to reduce attack surfaces and improve overall security posture. However, doing so must be done carefully to avoid any unintended downtime or functionality loss.

We will need to follow these steps:

Understand Current Permissions: Identify all permissions currently assigned.
Document the Application’s Needs: Determine which permissions are actually needed for the application to run.
Automate Testing: Develop a set of automated tests that mimic actual application behavior in production-like environments.
Re-evaluate and Adjust Permissions: Gradually remove unnecessary permissions based on test results, ensuring no functional breaks.

Here is the step-by-step process:

1. Understand Current Permissions

First, review the current IAM role policies associated with the instance. This involves listing all attached policies and inline policies to understand which permissions are in play.

BASH
aws iam list-attached-role-policies --role-name <instance-role>
2. Document the Application’s Needs

Next, we need a detailed understanding of what the application requires to function properly. This can be gathered through:

Code analysis: Inspecting code for AWS SDK calls and permissions used.
Logging analysis: Reviewing logs to identify actual permissions utilized during normal operations.
3. Automate Testing

To ensure that removing unnecessary permissions doesn’t break anything, we’ll create a set of automated tests:

PYTHON
import boto3

def test_functionality():
    client = boto3.client('sts', region_name='us-west-2')
    response = client.get_caller_identity()
    assert 'some-necessary-permission' in response['Arn']

# Run these tests in a CI/CD pipeline or directly in an isolated environment
4. Re-evaluate and Adjust Permissions

Start by removing the least critical permissions one by one, running automated tests after each change to ensure functionality remains intact.

BASH
aws iam detach-policy --role-name <instance-role> --policy-arn arn:aws:iam::<account-id>:policy/<policy-name>
5. Monitor and Feedback Loop

After making changes, closely monitor the application in a staging environment before rolling out to production. This ensures that any issues can be addressed without causing downtime.

#### Closing Summary

By following this systematic approach, we can systematically reduce the permissions of an instance role while ensuring its functionality remains intact. The key is careful and iterative testing combined with thorough documentation of what the application truly needs.

#### Common Mistakes

Not having a backup plan: Before making any changes, ensure you have the ability to revert if something goes wrong.
Failing to thoroughly document current permissions: Without this, it's hard to know which ones can be removed safely.
Overly aggressive removal: Too much too fast can lead to critical functionality breaks.

**Code:**

```python
import boto3

def test_functionality():
    client = boto3.client('sts', region_name='us-west-2')
    response = client.get_caller_identity()
    assert 'some-necessary-permission' in response['Arn']

# Run these tests in a CI/CD pipeline or directly in an isolated environment
```

**Example input / output:**

```

aws iam list-attached-role-policies --role-name example-role

CODE
Copy

Output:

{ "AttachedPolicies": [ { "PolicyArn": "arn:aws:iam::123456789012:policy/example-policy", "PolicyName": "example-policy" } ] }
```

