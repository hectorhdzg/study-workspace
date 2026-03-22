# RAG (Retrieval-Augmented Generation)

## What is RAG?

**RAG** combines a retrieval system with an LLM so the model can answer questions using external knowledge it wasn't trained on.

```
User Query
    ↓
[1. Embed Query] → vector
    ↓
[2. Search Vector DB] → top-k relevant chunks
    ↓
[3. Build Prompt] = system prompt + retrieved chunks + user query
    ↓
[4. LLM generates answer] grounded in retrieved context
    ↓
Response (with citations)
```

**Why RAG over fine-tuning?**
- No training cost — just index your documents
- Always up-to-date — re-index when data changes
- Traceable — you can cite sources
- Works with any LLM via API

---

## RAG Pipeline

### Step 1: Document Ingestion & Chunking

Split documents into small, overlapping chunks for embedding.

```python
def chunk_text(text, chunk_size=500, overlap=50):
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start += chunk_size - overlap
    return chunks

# Example
text = open("document.txt").read()
chunks = chunk_text(text)
print(f"{len(chunks)} chunks created")
```

```javascript
function chunkText(text, chunkSize = 500, overlap = 50) {
  const chunks = [];
  for (let start = 0; start < text.length; start += chunkSize - overlap) {
    chunks.push(text.slice(start, start + chunkSize));
  }
  return chunks;
}
```

| Chunking Strategy | Description | Best For |
|-------------------|-------------|----------|
| **Fixed-size** | Split by character/token count with overlap | Simple, general-purpose |
| **Sentence-based** | Split on sentence boundaries | Preserves sentence meaning |
| **Paragraph-based** | Split on `\n\n` | Structured documents |
| **Semantic** | Split when embedding similarity drops | Best quality, more complex |
| **Recursive** | Try paragraph → sentence → character | LangChain default, good balance |

**Chunk size trade-off:**
- Too small → loses context, retrieves noise
- Too large → dilutes relevant info, wastes tokens

Typical: **200-1000 tokens** with **10-20% overlap**.

---

### Step 2: Embedding & Indexing

```python
from openai import OpenAI

client = OpenAI()

def get_embeddings(texts):
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=texts
    )
    return [item.embedding for item in response.data]

# Embed all chunks
chunks = ["chunk 1 text...", "chunk 2 text...", ...]
vectors = get_embeddings(chunks)

# Store in vector database (e.g., Pinecone, Qdrant, pgvector)
```

```csharp
var embeddingOptions = new EmbeddingsOptions("text-embedding-3-small", chunks);
var response = await client.GetEmbeddingsAsync(embeddingOptions);
var vectors = response.Value.Data.Select(d => d.Embedding.ToArray()).ToList();
```

---

### Step 3: Retrieval

```python
def retrieve(query, top_k=5):
    query_vector = get_embeddings([query])[0]

    # Search vector DB (pseudo-code)
    results = vector_db.search(
        vector=query_vector,
        top_k=top_k,
        metric="cosine"
    )

    return [result.text for result in results]
```

### Step 4: Generation

```python
def rag_query(user_question):
    # Retrieve relevant chunks
    context_chunks = retrieve(user_question, top_k=5)
    context = "\n\n".join(context_chunks)

    # Build prompt
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content":
                "Answer the question based ONLY on the provided context. "
                "If the context doesn't contain the answer, say 'I don't know.' "
                "Cite the relevant section."},
            {"role": "user", "content":
                f"Context:\n{context}\n\nQuestion: {user_question}"}
        ],
        temperature=0,
    )

    return response.choices[0].message.content
```

---

## Advanced RAG Techniques

| Technique | Description |
|-----------|-------------|
| **Hybrid Search** | Combine vector similarity + keyword (BM25) search for better recall |
| **Re-ranking** | Use a cross-encoder to re-score results after initial retrieval |
| **Query Expansion** | Rephrase/expand user query to improve retrieval |
| **HyDE** | Generate a hypothetical answer, embed that, then search with it |
| **Metadata Filtering** | Filter by date, source, category before vector search |
| **Contextual Compression** | Trim retrieved chunks to only the relevant sentences |
| **Parent Document Retrieval** | Embed small chunks but return the full parent document |

---

## RAG Evaluation

| Metric | What It Measures |
|--------|-----------------|
| **Retrieval Recall** | Did we retrieve all relevant documents? |
| **Retrieval Precision** | Are the retrieved documents actually relevant? |
| **Faithfulness** | Is the answer supported by the retrieved context? |
| **Answer Relevancy** | Does the answer actually address the question? |
| **Hallucination Rate** | Does the model invent facts not in the context? |

---

## Common RAG Pitfalls

- **Poor chunking** → Relevant info split across chunks
- **No overlap** → Boundary context lost
- **Wrong embedding model** → Semantic mismatch between query and docs
- **Too few/many results** → Under-retrieval or context overload
- **Ignoring metadata** → Missing easy filters (date, category)
- **No re-ranking** → Top-k by cosine alone isn't always the best
