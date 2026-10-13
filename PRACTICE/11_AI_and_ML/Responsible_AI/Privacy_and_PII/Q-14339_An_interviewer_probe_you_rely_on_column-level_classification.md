---
type: "practice"
difficulty: "Deep"
---

# An interviewer probe: you rely on column-level classification, but the real risk is unstructured blobs and attachments. How do you architect PII discovery for documents images and PDFs at scale?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fine-tuned Models as Sensitive Artifacts](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Fine-tuned_Models_as_Sensitive_Artifacts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To handle PII in unstructured data like document images and PDFs, we need a robust architecture that combines both static content analysis and dynamic extraction techniques. This approach involves leveraging machine learning models, metadata inspection, and contextual understanding to identify sensitive information accurately at scale.

#### Triaging the Question

The interviewer is probing our ability to handle PII (Personally Identifiable Information) in unstructured data such as document images and PDFs. This requires a deep understanding of how to discover and classify PII beyond simple column-level classification techniques, which are more suitable for structured data. We need to architect a system that can scale effectively while ensuring high accuracy and minimal false positives.

#### Assumptions
Data Sources: Unstructured data in the form of document images (scanned documents) and PDFs.
Scope: The solution should be scalable, robust against common PII types like names, addresses, SSNs, etc., and must handle various file formats and structures.
Security Requirements: High emphasis on privacy and regulatory compliance.
Step-by-Step Solution
1. Data Ingestion

We start by ingesting the data from various sources (e.g., cloud storage, local directories). The ingestion process will involve regular scheduling to ensure that new or updated documents are processed promptly.

PYTHON
def ingest_documents(sources):
    documents = []
    for source in sources:
        files = os.listdir(source)
        for file in files:
            if is_image_or_pdf(file):  # Custom function to check file type
                document_data = extract_text_and_metadata(os.path.join(source, file))
                documents.append(document_data)
    return documents
2. Static Content Analysis

Static content analysis involves using pre-trained models or custom-trained models to recognize patterns that match known PII types. For this, we can leverage libraries like Tesseract for OCR (Optical Character Recognition) and NLP models from Hugging Face.

PYTHON
def static_content_analysis(text):
    pii_types = ['name', 'address', 'ssn']
    matches = []
    for type in pii_types:
        model = load_model(type)
        results = model.predict(text)
        matches.extend(results)
    return matches
3. Dynamic Extraction

Dynamic extraction involves analyzing the context and structure of the documents to identify PII types that may not be easily recognizable through static methods alone. This can include parsing tables, recognizing addresses in structured formats, or identifying names within specific contexts.

PYTHON
def dynamic_extraction(text):
    parsed_tables = parse_tables(text)
    contextual_matches = extract_contextual_pii(parsed_tables)
    return contextual_matches
4. Contextual Understanding and Metadata Inspection

To enhance accuracy, we can use metadata from the documents (e.g., file names, timestamps) and additional context provided by users or other systems.

PYTHON
def inspect_metadata(doc):
    metadata = get_document_metadata(doc)
    # Use metadata to refine PII detection
    return refined_pii_list

def get_document_metadata(doc):
    # Custom function to extract metadata from doc
    pass
5. Integration and Scaling

For scalability, we need a distributed system that can process multiple documents simultaneously. We can use Apache Kafka for event-driven architecture or AWS Lambda for serverless processing.

Ingest Documents
Static Content Analysis
Dynamic Extraction
Contextual Understanding andMetadata Inspection
Integration Layer (Kafka, AWSLambda)
6. Compliance and Security Measures

Ensure that the system complies with relevant regulations like GDPR or CCPA by implementing strong encryption, access controls, and audit logs.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, we need a comprehensive architecture that combines static content analysis, dynamic extraction techniques, and contextual understanding to accurately discover PII in unstructured data. This approach ensures high accuracy while maintaining scalability and security. Common mistakes include over-relying on static models without considering context or ignoring metadata inspection.

For follow-ups, consider discussing specific implementation details like model training, error handling, and performance optimization strategies for large-scale deployments.

**Comparison:**

| Document ID | File Type  | Text Content                                                                                                                                                      | Static PII         | Dynamic PII          | Metadata Inspection   | Final PII List                |
|-------------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|----------------------|------------------------|-------------------------------|
| 1           | PDF        | Customer: John Doe, Address: 123 Main St, City: Anytown, SSN: 123-45-6789                                                                                          | Name: John Doe     | Address: 123 Main St  | Timestamp: 2023-01-01   | Name: John Doe, Address: ... |
| 2           | JPG        | Recipient: Jane Smith, Phone: (555) 123-4567                                                                                                                      |                     |                      |                         |                              |

**Code:**

```python
def is_image_or_pdf(file_name):
    return file_name.endswith('.jpg') or file_name.endswith('.png') or file_name.endswith('.pdf')

def extract_text_and_metadata(path):
    # Custom extraction logic
    pass

def load_model(pii_type):
    # Load pre-trained model for specific PII type
    pass

def parse_tables(text):
    # Parse tables from text
    pass

def extract_contextual_pii(parsed_tables):
    # Extract contextual matches based on parsed tables
    pass

def get_document_metadata(doc):
    # Custom function to extract metadata
    pass
```

**Example input / output:**

```text
documents = ingest_documents(["/path/to/document/source"])
for doc in documents:
    static_matches = static_content_analysis(doc['text'])
    dynamic_matches = dynamic_extraction(doc['text'])
    metadata_inspection_result = inspect_metadata(doc)

    final_pii_list = merge_and_refine_matches(static_matches, dynamic_matches, metadata_inspection_result)
    print(f"PII Matches for {doc['filename']}: {final_pii_list}")
```

**Data:**

```json
{
  "documents": [
    {
      "filename": "invoice_123456.pdf",
      "text": "Customer: John Doe, Address: 123 Main St, City: Anytown, SSN: 123-45-6789"
    },
    {
      "filename": "contract_007.jpg",
      "text": "Recipient: Jane Smith, Phone: (555) 123-4567"
    }
  ]
}
```

**Flow:**

1. Ingest Documents
2. Static Content Analysis
3. Dynamic Extraction
4. Contextual Understanding andMetadata Inspection
5. Integration Layer (Kafka, AWSLambda)
6. Evaluation & Refinement

