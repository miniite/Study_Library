---
type: "practice"
difficulty: "Basic"
---

# How would you read text out of a PDF in Python for a RAG pipeline, and which library would you reach for?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Compressed File Handling](<../../../../TOOLS/Python/Python_Language/Compressed_File_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To read text from a PDF file in Python for a Retrieval-Augmented Generation (RAG) pipeline, the core task is to extract structured or unstructured text data from the PDF document. A simple and effective library for this purpose is PyPDF2 for basic tasks and pdfplumber for more advanced operations like handling tables and images within the PDF.

First, let's understand what these libraries offer and why we would choose one over another.

**Comparison:**

```
Library	Features	Best For
PyPDF2	Basic text extraction, simple API	Quick setup and easy to use for basic tasks
pdfplumber	Advanced features (tables, images)	More complex PDF parsing needs
```

**Code:**

```python
# Importing PyPDF2 for basic text extraction
import PyPDF2

# Importing pdfplumber for more advanced PDF parsing
import pdfplumber
```

**Example input / output:**

```

Using PyPDF2:

PYTHON
Copy
def extract_text_from_pdf_pypdf2(pdf_path):
    # Open the file and get the file object
    pdf_file = open(pdf_path, 'rb')

    # Create a PDF reader object
    pdf_reader = PyPDF2.PdfReader(pdf_file)

    text = ''
    for page_num in range(len(pdf_reader.pages)):
        page_obj = pdf_reader.pages[page_num]
        text += page_obj.extract_text()

    # Close the file object
    pdf_file.close()
    return text

# Example usage
pdf_path = 'example.pdf'
text_content = extract_text_from_pdf_pypdf2(pdf_path)
print(text_content)

Using pdfplumber:

PYTHON
Copy
def extract_text_from_pdf_pdfplumber(pdf_path):
    # Open the file and get the file object
    with pdfplumber.open(pdf_path) as pdf:
        text = ''
        for page in pdf.pages:
            text += page.extract_text()

    return text

# Example usage
pdf_path = 'example.pdf'
text_content = extract_text_from_pdf_pdfplumber(pdf_path)
print(text_content)
```

**Data:**

```text
For demonstration, assume the following simple structure of a PDF document with multiple pages and some text content on each page.
```

**Flow:**

1. Rendering diagram…

