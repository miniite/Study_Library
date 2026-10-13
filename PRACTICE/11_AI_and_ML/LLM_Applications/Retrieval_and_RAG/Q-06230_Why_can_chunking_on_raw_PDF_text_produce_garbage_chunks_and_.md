---
type: "practice"
difficulty: "Intermediate"
---

# Why can chunking on raw PDF text produce garbage chunks and how would you notice it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Structure Aware Chunking](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Structure_Aware_Chunking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When using chunking techniques to process raw PDF text, one common issue is producing "garbage chunks" due to improper handling of formatting and structure. These issues can be identified by evaluating the semantic coherence of the generated chunks.

#### Triaging the Question

The question probes your understanding of natural language processing (NLP) and data preprocessing in AI systems, specifically focusing on the challenges faced when extracting meaningful information from unstructured text like PDFs. The core idea is to ensure that the chunking process maintains the context and meaning of the original content.

#### Assumptions
& First-Principles Reasoning
Raw PDF Text: Typically, raw PDF text retains formatting elements such as headers, footers, images, and tables.
Chunking Process: The goal is to break down the document into smaller, meaningful segments (chunks) for further processing or analysis.
Garbage Chunks: These are chunks that do not carry relevant information or make little sense in context.
Step-by-Step Solution

Understanding PDF Structure:

Raw PDFs often contain non-textual elements like images and tables, which can disrupt the natural flow of text.
Headers and footers might be included as part of the raw text, leading to chunks that lack context or are semantically irrelevant.

Preprocessing Steps:

Text Extraction: Use tools like Apache PDFBox or PyMuPDF (PyMuPDF) to extract only the textual content from the PDF.
Tokenization: Break down the extracted text into words and sentences for further processing.
Formatting Removal: Clean up formatting elements that do not add value, such as removing header and footer sections.

Chunking Algorithms:

Word-Based Chunking: Divide text based on logical word breaks (e.g., sentence boundaries).
Sentence-Based Chunking: Create chunks from complete sentences or sentence fragments.
Topic-Based Chunking: Group related content into topics, which may involve topic modeling techniques like LDA.

Evaluating Semantic Coherence:

Contextual Analysis: Ensure that each chunk is contextually relevant and meaningful.
Stop Words Removal: Remove common stop words to focus on key information.
Token Frequency Analysis: Analyze the frequency of tokens within chunks to identify noise or irrelevant content.

#### Example Scenario

#### Cons
ider a PDF containing both text and table data. Without proper preprocessing, extracting chunks might result in garbage like "Page 13 of 20" (header) followed by "Table of contents" (footer), which is irrelevant for downstream processing.

#### Closing Summary

By understanding the challenges associated with raw PDFs and implementing robust preprocessing steps, you can effectively identify and mitigate the production of garbage chunks. Evaluating semantic coherence is crucial to ensure that only meaningful segments are generated.

#### Common Mistakes
& Follow-Ups
Ignoring Formatting: Not properly removing or handling non-textual elements like headers and footers.
Overprocessing: Overly aggressive removal of stop words, leading to loss of valuable context.
Inadequate Evaluation: Failing to thoroughly test the semantic coherence of generated chunks.

Would you need any code examples for a specific part of this process?

**Comparison:**

```
Page Number	Text Chunk
13	Page 13 of 20
	Table of contents
```

**Code:**

```python
# Example Python Code for Text Extraction from PDF using PyMuPDF

import fitz  # PyMuPDF library

def extract_text_from_pdf(pdf_path):
    document = fitz.open(pdf_path)
    text = ""
    for page_num in range(len(document)):
        page = document[page_num]
        blocks = page.get_text("dict")["blocks"]
        for block in blocks:
            if "lines" in block:
                for line in block["lines"]:
                    for span in line["spans"]:
                        # Extracting text from spans
                        text += span["text"] + " "
    return text

# Example usage
pdf_text = extract_text_from_pdf("example.pdf")
print(pdf_text)
```

**Example input / output:**

```
Input: Raw text from a PDF containing headers, footers, and tables.
Output: Processed chunks that are contextually relevant.
PLAINTEXT
Copy
Input:
Page 13 of 20
Table of contents

Output (after processing):
"Table of contents"
```

**Data:**

```json
{
  "headers": ["Page Number", "Text Chunk"],
  "data": [
    {"13 of 20": "Page 13 of 20"},
    {1: "Table of contents"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Extract text from PDF
5. Is chunk contextually relevant?
6. Output processed chunks
7. Remove non-relevant content,e.g., headers/footers
8. Re-evaluate chunk

