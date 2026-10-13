---
type: "practice"
difficulty: "Deep"
---

# How would you design a pipeline to enforce PII masking and data-residency rules for EU vs India users in the same flow?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pipeline Design Patterns](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need for enforcing PII masking and data-residency rules for EU vs India users within the same data pipeline, we must first understand that this is a complex problem involving multiple layers of security and compliance requirements. We will break it down into key steps: identifying user regions, applying appropriate masking techniques, ensuring data residency, and logging all operations.

First, let's clarify what we're trying to achieve:

PII Masking: Ensuring Personally Identifiable Information (PII) is obscured or anonymized.
Data Residency Rules: Ensuring that data is stored within the legal jurisdiction of the user (EU vs India).

#### Triaging the Question

The question asks us to design a pipeline that handles both EU and Indian users while adhering to their respective rules. We need to ensure:

User region identification.
PII masking based on user region.
Data residency compliance.
Logging of all operations for audit purposes.

#### Assumptions
User Region Identification: The system has a mechanism to determine the user's geographical location or region (e.g., IP address, user metadata).
PII Fields: We have an understanding of which fields are considered PII and need masking.
Data Storage Locations: Known locations for storing EU data versus Indian data.
Step-by-Step Solution

User Region Identification:

Use a geo-location service or analyze the IP address to determine if the user is in the EU or India.

PII Masking:

Implement a masking function that can handle different types of PII fields (e.g., names, addresses, phone numbers).
Apply different masking techniques based on the region: more stringent masking for EU data to comply with GDPR.

Data Residency Compliance:

Ensure that data from EU users is stored in compliant data centers within the EU.
Similarly, ensure that data from Indian users is stored in compliant data centers within India.

Logging and Auditing:

Log all operations related to PII handling, including masking, storage, and access.
Ensure logs are retained for a sufficient period (e.g., 7 years) as per GDPR or other regulations.
Detailed Solution

User Region Identification:

PYTHON
import geolite2

def get_user_region(ip_address):
    reader = geolite2.reader()
    location_info = reader.get(ip_address)
    if location_info and 'country' in location_info:
        region = location_info['country']['iso_code']
    else:
        region = 'unknown'
    geolite2.close()
    return region

PII Masking:

PYTHON
def mask_pii(field, pii_type):
    if pii_type == 'EU':
        # More stringent masking for EU data (e.g., full name masked to initials)
        if field.startswith('name'):
            return f"{field[0]}. {field.split()[1]}"
    elif pii_type == 'India':
        # Less stringent masking for India data
        if field.startswith('address'):
            return "XXXX-XXXX, XXXXX, XX"
    return field

def apply_masking(record):
    user_region = get_user_region(record['ip_address'])
    record['name'] = mask_pii(record['name'], user_region)
    record['address'] = mask_pii(record['address'], user_region)
    # Additional PII fields can be masked similarly

Data Residency Compliance:

Use a data storage service that supports regional storage (e.g., AWS S3 with S3 Outposts).
Configure the pipeline to route data based on the determined region.

Logging and Auditing:

PYTHON
import logging

def log_operation(operation):
    logger = logging.getLogger(__name__)
    logger.info(f"Operation: {operation}")

def process_record(record):
    user_region = get_user_region(record['ip_address'])
    pii_masked_record = apply_masking(record)
    log_operation(f"Masked record for region: {user_region}")
    # Store masked data in the appropriate regional storage

#### Closing Summary
and Common Mistakes/Follow-Ups

By following this approach, we can ensure that PII is properly masked based on user regions while adhering to strict data residency rules. Common mistakes include:

Overlooking the need for logging and auditing.
Failing to apply appropriate masking techniques for different regions.
Not considering edge cases (e.g., users with multiple regions).

For follow-ups, we might explore:

Cost Optimization: Ensuring that regional storage costs are minimized without compromising compliance.
Performance Impact: Evaluating the performance impact of additional operations like region determination and logging.

**Comparison:**

```
User Region	PII Field	Masking Technique
EU	name	Initials
EU	address	XXXX-XXXX, XXXXX, XX
India	name	Full Name
India	address	XXXX-XXXX, XXXXX, XX
```

**Code:**

```python
import geolite2

def get_user_region(ip_address):
    reader = geolite2.reader()
    location_info = reader.get(ip_address)
    if location_info and 'country' in location_info:
        region = location_info['country']['iso_code']
    else:
        region = 'unknown'
    geolite2.close()
    return region

def mask_pii(field, pii_type):
    if pii_type == 'EU':
        # More stringent masking for EU data (e.g., full name masked to initials)
        if field.startswith('name'):
            return f"{field[0]}. {field.split()[1]}"
    elif pii_type == 'India':
        # Less stringent masking for India data
        if field.startswith('address'):
            return "XXXX-XXXX, XXXXX, XX"
    return field

def apply_masking(record):
    user_region = get_user_region(record['ip_address'])
    record['name'] = mask_pii(record['name'], user_region)
    record['address'] = mask_pii(record['address'], user_region)
    # Additional PII fields can be masked similarly
```

**Example input / output:**

```text
record = {
    "ip_address": "213.195.70.82",
    "name": "John Doe",
    "address": "123 Main St, New York, NY"
}

pii_masked_record = apply_masking(record)
print(pii_masked_record)  # {'name': 'J. Doe', 'address': 'XXXX-XXXX, XXXXX, XX'}
```

**Data:**

```json
{
  "record": {
    "ip_address": "213.195.70.82",
    "name": "John Doe",
    "address": "123 Main St, New York, NY"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Region Identification
6. Is User in EU?
7. Apply Strict Masking
8. Is User in India?
9. Apply Less Stringent Masking
10. No PII Fields to Mask
11. Store Data in EU Storage
12. Store Data in Indian Storage
13. Log Operation

