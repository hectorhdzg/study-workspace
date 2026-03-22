# Vector Databases & Embeddings

## Why Vector Databases?

Traditional databases search by exact match or keyword. **Vector databases** search by _meaning_ — finding the most semantically similar items using embedding vectors.

```
"How do I reset my password?"  →  [0.12, -0.34, 0.78, ...]  (1536 dims)
"I forgot my login credentials" → [0.11, -0.32, 0.76, ...]  (very similar!)
"What's your return policy?"    → [0.89, 0.15, -0.23, ...]  (very different)
```

---

## Popular Vector Databases

| Database | Type | Hosting | Best For |
|----------|------|---------|----------|
| **Pinecone** | Managed cloud | SaaS | Production, zero-ops |
| **Qdrant** | Open-source / Cloud | Self-host or cloud | Flexible, Rust-based |
| **Weaviate** | Open-source / Cloud | Self-host or cloud | Hybrid search built-in |
| **Milvus** | Open-source | Self-host | Large-scale, distributed |
| **Chroma** | Open-source | In-process | Prototyping, local dev |
| **pgvector** | PostgreSQL extension | Existing Postgres | When you already use Postgres |
| **Redis** | Vector module | Self-host or cloud | Fast, familiar |

---

## Embedding Models

| Model | Provider | Dimensions | Strengths |
|-------|----------|-----------|-----------|
| text-embedding-3-small | OpenAI | 1536 | Good quality, affordable |
| text-embedding-3-large | OpenAI | 3072 | Best quality (OpenAI) |
| voyage-3 | Voyage AI | 1024 | Strong for code & retrieval |
| BGE / GTE | Open-source | 768-1024 | Free, self-hostable |
| Cohere Embed v3 | Cohere | 1024 | Built-in search type param |

---

## Usage Examples

### Chroma (Local / Prototyping)

```python
import chromadb

client = chromadb.Client()
collection = client.create_collection("docs")

# Add documents (auto-embeds with default model)
collection.add(
    ids=["doc1", "doc2", "doc3"],
    documents=[
        "Python is a programming language",
        "JavaScript runs in the browser",
        "Machine learning uses statistical models"
    ]
)

# Query by meaning
results = collection.query(
    query_texts=["web development language"],
    n_results=2
)
print(results["documents"])
# [['JavaScript runs in the browser', 'Python is a programming language']]
```

### pgvector (PostgreSQL)

```sql
-- Enable extension
CREATE EXTENSION vector;

-- Create table with embedding column
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(1536)  -- 1536-dimensional vector
);

-- Insert a document with its embedding
INSERT INTO documents (content, embedding)
VALUES ('Hello world', '[0.1, 0.2, ..., 0.3]');

-- Find 5 most similar documents (cosine distance)
SELECT content, 1 - (embedding <=> $1::vector) AS similarity
FROM documents
ORDER BY embedding <=> $1::vector
LIMIT 5;
```

### Pinecone (Managed)

```python
from pinecone import Pinecone

pc = Pinecone(api_key="your-key")
index = pc.Index("my-index")

# Upsert vectors
index.upsert(vectors=[
    {"id": "doc1", "values": [0.1, 0.2, ...], "metadata": {"source": "faq"}},
    {"id": "doc2", "values": [0.3, 0.4, ...], "metadata": {"source": "docs"}},
])

# Query with metadata filter
results = index.query(
    vector=[0.1, 0.2, ...],
    top_k=5,
    filter={"source": {"$eq": "faq"}},
    include_metadata=True,
)
```

---

## Indexing Strategies

| Index Type | How It Works | Trade-off |
|-----------|-------------|-----------|
| **Flat (Brute Force)** | Compare query to every vector | 100% recall, slow at scale |
| **IVF (Inverted File)** | Cluster vectors, only search nearby clusters | Fast, slight recall loss |
| **HNSW** (Hierarchical Navigable Small World) | Graph-based, multi-layer navigation | Fast, high recall, more memory |
| **PQ** (Product Quantization) | Compress vectors for memory efficiency | Very compact, lower accuracy |

**HNSW is the most popular choice** for production — best balance of speed and accuracy.

---

## Best Practices

- **Normalize vectors** before storing if using cosine similarity
- **Choose dimensions wisely** — higher dims = more accuracy but more cost/latency
- **Batch upserts** — don't insert one vector at a time
- **Use metadata filters** — narrow search space before vector similarity
- **Monitor recall** — track if users are getting relevant results
- **Re-embed when switching models** — embeddings from different models are incompatible
