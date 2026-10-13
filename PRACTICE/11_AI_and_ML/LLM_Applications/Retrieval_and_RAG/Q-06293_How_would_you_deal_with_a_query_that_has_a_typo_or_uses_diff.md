---
type: "practice"
difficulty: "Intermediate"
---

# How would you deal with a query that has a typo or uses different wording than the document?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Query Rewriting and Fuzzy Retrieval](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Query_Rewriting_and_Fuzzy_Retrieval.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a query that has a typo or uses different wording, our goal is to accurately retrieve relevant information despite these discrepancies. This challenge primarily arises in natural language processing (NLP) contexts where queries might be phrased differently from the documents stored in our database. The solution involves several steps: preprocessing the input and the document text, leveraging advanced NLP techniques, and employing a robust retrieval strategy.

Firstly, it's crucial to understand that this problem is common in many applications such as chatbots, search engines, or content-based recommendation systems. To effectively handle these cases, we need to break down the query into smaller meaningful units, which are then used for comparison against stored documents. This process is known as chunking.

Secondly, we should consider the types of retrieval strategies that can be employed. The most common ones include exact matching, fuzzy matching, and semantic matching.

Let's delve into a practical solution from first principles:

Preprocessing: Start by cleaning both the query and the document text. This involves lowercasing, removing punctuation, stop words, and stemming/lemmatizing to normalize the text. For instance:

PYTHON
import nltk
from nltk.corpus import stopwords

def preprocess(text):
    tokens = nltk.word_tokenize(text.lower())
    stop_words = set(stopwords.words('english'))
    filtered_tokens = [token for token in tokens if token.isalpha() and token not in stop_words]
    return ' '.join(filtered_tokens)

Chunking: Break the text into meaningful units such as sentences, phrases, or even individual words depending on the context. For example:

PYTHON
def chunk(text):
    return nltk.sent_tokenize(text)

Fuzzy Matching: Use techniques like Levenshtein distance (edit distance) to compare the query with each sentence in the document. This can help identify sentences that are close but not exact matches.

PYTHON
from fuzzywuzzy import process

def fuzzy_match(query, doc_chunk):
    return process.extractOne(query, [doc_chunk])[1]

Semantic Matching: Use NLP models like BERT or Sentence-BERT to convert sentences into embeddings and then compute cosine similarity between the query embedding and document sentence embeddings.

PYTHON
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')
query_embedding = model.encode(query)
doc_embeddings = [model.encode(chunk) for chunk in documents]
similarities = [cosine_similarity(query_embedding, embedding)[0] for embedding in doc_embeddings]

Retrieval Strategy: Once we have computed the similarity scores, rank the sentences or chunks based on these scores and return the top-k results as potential matches.

Final Output: Present the ranked list of sentences or chunks that best match the query.

PYTHON
def retrieve_top_k(query, documents):
    preprocessed_query = preprocess(query)
    chunks = chunk(preprocessed_query)
    similarities = [fuzzy_match(chunk, document) for chunk, document in zip(chunks, documents)]
    ranked_chunks = sorted(zip(similarities, chunks), reverse=True)[:5]
    return [chunk[1] for chunk in ranked_chunks]

This solution is a mix of preprocessing to clean and normalize the text, chunking to break down into meaningful units, fuzzy matching to handle typos, semantic matching for more advanced word meaning analysis, and finally, a retrieval strategy to rank and present relevant results.

