---
type: "practice"
difficulty: "Intermediate"
---

# How would you make sure a user's private uploaded documents are only searchable by that same user in a multi-user RAG app?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Presigned URLs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Presigned_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a user’s private uploaded documents remain searchable exclusively by that same user within a multi-user Retrieval-Augmented Generation (RAG) application, we need to implement stringent access controls and secure storage mechanisms. At an intermediate level, this involves understanding the cloud fundamentals of authentication, authorization, encryption at rest, and in transit. The key steps include designing a robust data pipeline with appropriate access policies.

#### Triaging the Question

The question is fundamentally asking about ensuring the privacy and confidentiality of user-uploaded documents within a multi-user RAG application. It probes our knowledge on secure data storage practices and understanding how to implement fine-grained access controls in cloud environments. The core assumptions are that we have a multi-tenant architecture with multiple users, each having their own set of private documents.

#### First-Principles Solution
Authentication: Use robust user authentication mechanisms like OAuth2 or JWT (JSON Web Tokens) to ensure only authenticated and authorized users can access the RAG application.
Authorization: Implement role-based access control (RBAC) where each user is assigned roles that determine their permissions, such as read-only access for certain documents.
Encryption: Encrypt the documents both at rest and in transit using strong encryption standards like AES-256.
Access Policies: Configure S3 bucket policies or equivalent cloud storage policies to restrict document access based on user credentials.
Detailed Steps

User Authentication

Utilize a secure authentication service such as AWS Cognito, Azure Active Directory (AAD), or Google Identity Platform.
Users log in using their credentials, and upon successful verification, they receive an access token that can be used to authenticate subsequent API calls.

Role-Based Access Control (RBAC)

Define roles for users based on their permissions within the RAG app.
For example:
YAML
roles:
  - name: user
    policies:
      - documents:GetPrivateDocuments
      - documents:ListPublicDocuments
Ensure that only users with the appropriate role can access private documents.

Encryption

Use server-side encryption (SSE) to protect documents at rest.
For documents uploaded via an API, use SSE-KMS (Key Management Service) provided by AWS or Azure KMS for end-to-end encryption.
Implement client-side encryption where the user's device encrypts the document before uploading it.

Access Policies

Configure S3 bucket policies to restrict access to private documents based on the user’s credentials.

#### Example
policy:
JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnauthenticatedUserAccessToPrivateDocuments",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestTag/username": "<user-identifier>"
        }
      }
    },
    {
      "Sid": "AllowPrivateDocumentAccessToAuthenticatedUsers",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/private/*",
      "Condition": {
        "StringEquals": {
          "aws:RequestTag/username": "<user-identifier>"
        }
      }
    }
  ]
}

API Security

Ensure that all API endpoints are secured with proper authentication and authorization.
Use HTTPS to encrypt data in transit.

#### Closing Summary

By implementing a combination of robust authentication, role-based access control, encryption mechanisms, and appropriate S3 bucket policies, we can ensure that user-uploaded documents are only searchable by the same user. This approach not only protects the privacy and confidentiality of documents but also adheres to best practices in cloud security.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to use strong encryption standards or not properly configuring it.
Mistake 2: Not implementing fine-grained access controls, leading to broader-than-necessary permissions.

#### Follow-Up
Discussing how to handle edge cases like forgotten passwords, user account deactivation, and ensuring compliance with data protection regulations.

**Comparison:**

```
Role	Permissions
user	- GetPrivateDocuments <br> - ListPublicDocuments
```

**Code:**

```python
# Example of a simple JWT-based authentication in Python using Flask
from flask import Flask, request, jsonify
import jwt

app = Flask(__name__)

# Dummy secret key for demonstration purposes; should be securely managed
SECRET_KEY = 'your_secret_key'

@app.route('/login', methods=['POST'])
def login():
    # Assume user credentials are checked here
    user_id = 123  # Example user ID
    token = jwt.encode({'user_id': user_id}, SECRET_KEY, algorithm='HS256')
    return jsonify({'token': token})

@app.route('/documents/private/<document_id>', methods=['GET'])
def get_private_document(document_id):
    token = request.headers.get('Authorization').split()[1]
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        # Check if the user is allowed to access this document
        return jsonify({'content': 'Private document content'})
    except jwt.ExpiredSignatureError:
        return jsonify({'error': 'Token has expired'}), 401
```

**Example input / output:**

```text
Input (API call to get a private document):
POST /login
{
  "username": "john_doe",
  "password": "secure_password"
}

Response:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMiwiaWF0IjoxNjY3NzkxNTk4fQ.hF7n8PZV3pBmLgk4uTtEjgUeFzK956dXGyWJ1i"
}

Input (API call to get a private document with valid token):
GET /documents/private/123
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMiwiaWF0Ijo2NjcnNzkxNTk4fQ.hF7n8PZV3pBmLgk4uTtEjgUeFzK956dXGyWJ1i

Response:
{
  "content": "Private document content"
}
```

**Data:**

```yaml
# Example of RBAC configuration for a RAG app
roles:
  - name: user
    policies:
      - documents:GetPrivateDocuments
      - documents:ListPublicDocuments

users:
  - id: 123
    roles: [user]
```

**Flow:**

1. User Authentication
2. Generate JWT Token
3. API Call to Get PrivateDocument
4. Validate JWT Token
5. Access Allowed? Yes -->F[Return Document Content];No --> G[Error: Unauthorized]

