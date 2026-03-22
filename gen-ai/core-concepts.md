# Core Concepts

## The Transformer Architecture

The **Transformer** (2017, "Attention Is All You Need") replaced RNNs/LSTMs as the dominant architecture for sequence modeling. It processes all tokens in parallel using **self-attention**.

### Key Components

```
Input → [Tokenization] → [Embedding + Positional Encoding]
      → [Encoder Blocks × N] → [Decoder Blocks × N]
      → [Linear + Softmax] → Output Token
```

| Component | Purpose |
|-----------|---------|
| **Tokenizer** | Splits text into tokens (subwords, not full words) |
| **Embedding Layer** | Maps tokens to dense vectors (e.g., 768 or 1536 dimensions) |
| **Positional Encoding** | Adds position information (transformers have no inherent order) |
| **Self-Attention** | Each token attends to every other token — captures context |
| **Feed-Forward Network** | Applies non-linear transformation per position |
| **Layer Normalization** | Stabilizes training |

---

## Attention Mechanism

**Self-attention** lets the model weigh how much each token should "attend to" every other token.

$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

- **Q (Query):** What am I looking for?
- **K (Key):** What do I contain?
- **V (Value):** What information do I provide?
- **$d_k$:** Dimension of key vectors (scaling factor prevents large dot products)

### Multi-Head Attention

Run attention multiple times in parallel with different learned projections, then concatenate:

```
MultiHead(Q, K, V) = Concat(head_1, ..., head_h) · W_O
where head_i = Attention(Q·W_Qi, K·W_Ki, V·W_Vi)
```

This lets the model capture different types of relationships (syntactic, semantic, positional) simultaneously.

---

## Tokenization

Tokenizers convert text into integer token IDs that the model understands.

| Method | Description | Example |
|--------|-------------|---------|
| **BPE** (Byte Pair Encoding) | Merges frequent character pairs iteratively | GPT-2, GPT-4 |
| **WordPiece** | Similar to BPE, used by BERT | `"playing"` → `["play", "##ing"]` |
| **SentencePiece** | Language-agnostic, operates on raw text | Used by LLaMA, T5 |

```python
# Using tiktoken (OpenAI's tokenizer)
import tiktoken

enc = tiktoken.encoding_for_model("gpt-4")
tokens = enc.encode("Hello, how are you?")
print(tokens)        # [9906, 11, 1268, 527, 499, 30]
print(len(tokens))   # 6 tokens

# Decode back
text = enc.decode(tokens)
print(text)          # "Hello, how are you?"
```

**Key insight:** 1 token ≈ 4 characters ≈ ¾ of a word in English. Token count matters because:
- Models have a **context window** (max tokens per request)
- You pay per token (input + output) with API providers
- Longer prompts = higher latency

---

## Embeddings

**Embeddings** are dense vector representations of text. Semantically similar text produces vectors that are close together in high-dimensional space.

```python
from openai import OpenAI
client = OpenAI()

response = client.embeddings.create(
    model="text-embedding-3-small",
    input="What is machine learning?"
)

vector = response.data[0].embedding
print(len(vector))  # 1536 dimensions
```

```csharp
// Using Azure OpenAI SDK
var client = new OpenAIClient(new Uri(endpoint), new AzureKeyCredential(key));
var options = new EmbeddingsOptions("text-embedding-3-small",
    new[] { "What is machine learning?" });
var response = await client.GetEmbeddingsAsync(options);
float[] vector = response.Value.Data[0].Embedding.ToArray();
// 1536 dimensions
```

### Similarity Metrics

| Metric | Formula | Range | Best For |
|--------|---------|-------|----------|
| **Cosine Similarity** | $\frac{A \cdot B}{\|A\| \|B\|}$ | [-1, 1] | Most text similarity tasks |
| **Dot Product** | $A \cdot B$ | (-∞, +∞) | When magnitude matters |
| **Euclidean Distance** | $\|A - B\|_2$ | [0, +∞) | Clustering |

```python
import numpy as np

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))
```

---

## Encoder vs Decoder Models

| Architecture | What It Does | Examples | Use Cases |
|-------------|-------------|----------|-----------|
| **Encoder-only** | Bidirectional — sees full input at once | BERT, RoBERTa | Classification, NER, embeddings |
| **Decoder-only** | Autoregressive — generates one token at a time | GPT-4, LLaMA, Claude | Text generation, chat, code |
| **Encoder-Decoder** | Encode input, then decode output | T5, BART | Translation, summarization |

**Most modern LLMs are decoder-only** — they generate text left-to-right, predicting the next token given all previous tokens.

---

## Key Terms Quick Reference

| Term | Definition |
|------|-----------|
| **Context Window** | Maximum number of tokens a model can process in one request |
| **Temperature** | Controls randomness (0 = deterministic, 1 = creative) |
| **Top-p (Nucleus Sampling)** | Sample from the smallest set of tokens whose probability sums to p |
| **Logits** | Raw, unnormalized scores output by the model before softmax |
| **Perplexity** | How "surprised" the model is by the text — lower = better |
| **Inference** | Running a trained model to generate predictions |
| **Fine-tuning** | Continuing training on a specific dataset |
| **RLHF** | Reinforcement Learning from Human Feedback — aligns model to preferences |
