---
type: "practice"
difficulty: "Expert"
---

# How do you architect the platform so a new business unit can be fully isolated (blast-radius, cost, access) yet still share golden reference data — multi-tenancy strategy at the org level?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Late and Out-of-order Data Strategy](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Late_and_Out-of-order_Data_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a data platform that enables full isolation of a new business unit while sharing common golden reference data, we need to focus on multi-tenant strategies that balance blast radius, cost, and access control effectively. The core idea is to design the platform so it supports both isolation and collaboration through robust tenant management, secure data sharing mechanisms, and optimized resource allocation.

At a high level, our approach involves:

Multi-Tenancy at the Org Level: Designing the platform to support multiple business units as tenants.
Isolation Mechanisms: Ensuring that each business unit operates in its own environment with controlled access and limited visibility into other units.
Golden Reference Data Management: Establishing a strategy for sharing essential data while maintaining control over how it's used by different tenants.

Let’s break down the solution step-by-step:

#### Triaging the Question

The question is asking about how to design a multi-tenant architecture that can handle both isolation and collaboration within an organization. The key aspects are:

Isolation: Ensuring that data and resources of one business unit do not interfere with others.
Access Control: Managing who has access to what data and services.
Cost Management: Ensuring the platform is cost-effective for each tenant while providing necessary services.

#### First Principles Solution

Tenant Segmentation:

Each business unit (tenant) should have its own logical and physical segmentation.
Logical segmentation could involve namespaces, schema organization, or tagging in data storage and processing layers.
Physical segmentation can be achieved through dedicated clusters or virtual private clouds.

Access Control Strategy:

Implement role-based access control (RBAC) to manage user roles and permissions across the platform.
Use fine-grained controls like attribute-based access control (ABAC) for more complex permission models.
Leverage authentication services such as OAuth or SAML for secure identity management.

Data Sharing Mechanisms:

Define a clear policy for sharing golden reference data. This could involve creating read-only views of shared datasets.
Implement data fabric technologies that allow controlled and governed access to these shared datasets.

Resource Management:

Use containerization and cloud-native services like Kubernetes or AWS ECS to manage resources efficiently.
Utilize cost management tools provided by the cloud provider (e.g., AWS Cost Explorer) to monitor and control costs for each tenant.

Monitoring and Auditing:

Implement comprehensive monitoring and logging to detect any unauthorized access or misuse of shared data.
Set up automated alerts based on predefined policies.

#### Closing Summary

By implementing a multi-tenant strategy that balances isolation, access control, and cost management, we can ensure that each business unit in the organization has its own secure environment while still benefiting from shared resources and golden reference data. Key considerations include logical and physical segmentation, robust access controls, efficient resource utilization, and comprehensive monitoring.

#### Common Mistakes/Follow-Ups
Common Mistake: Failing to implement strong isolation mechanisms, which can lead to accidental or malicious data leakage.

#### Follow-Up
Discussing how to handle edge cases like cross-tenant data dependencies and ensuring that tenant boundaries are clear and well-defined.

**Comparison:**

```
Tenant	Roles
BU1	Admin, Analyst
BU2	Admin
Shared Dataset	Access Levels
101	BU2 - Read Only
	BU3 - Write
```

**Code:**

```python
# Example of a simple RBAC setup using Flask and SQLAlchemy
from flask import Flask, request
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///tenants.db'
db = SQLAlchemy(app)

class Tenant(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), unique=True, nullable=False)
    roles = db.relationship('Role', backref='tenant')

class Role(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    role_name = db.Column(db.String(80), unique=True, nullable=False)
    tenant_id = db.Column(db.Integer, db.ForeignKey('tenant.id'))

@app.route('/data', methods=['GET'])
def get_data():
    # Example: Check if the user has permission to access data
    username = request.authorization['username']
    role = Role.query.join(Tenant).filter_by(name=username).first()
    if role:
        return "Data accessed successfully!"
    else:
        return "Access denied", 403

if __name__ == '__main__':
    db.create_all()
    app.run(debug=True)
```

**Example input / output:**

```text
Input:
- Tenant request for data access (with authentication)
Output:
- Access granted or denied based on RBAC rules and shared dataset policies

Request: GET /data
Response: Data accessed successfully!
```

**Data:**

```json
{
  "tenant": {
    "id": 1,
    "name": "BusinessUnitA",
    "roles": [
      { "role_name": "Admin" },
      { "role_name": "Analyst" }
    ]
  },
  "data_sharing_policy": {
    "shared_dataset_id": 101,
    "access_levels": [
      { "tenant_id": 2, "read_only": true },
      { "tenant_id": 3, "write_access": true }
    ]
  }
}
```

**Flow:**

1. User Authentication
2. Tenant Identification
3. Access Control (RBAC)
4. Data Access Request Handling
5. Data Retrieval/Access Decision
6. Success Response
7. Denied Response

