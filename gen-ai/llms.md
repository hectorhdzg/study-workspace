# Large Language Models (LLMs)

## How LLMs Work

An LLM is a **decoder-only transformer** trained on massive text corpora to predict the next token. Despite this simple objective, scale and data produce emergent capabilities like reasoning, coding, and instruction following.

### Training Pipeline

```
1. Pre-training     → Learn language from internet-scale text (trillions of tokens)
                      Objective: predict next token
                      Cost: millions of dollars, weeks on thousands of GPUs

2. Supervised       → Fine-tune on instruction/response pairs
   Fine-Tuning        (SFT — makes the model follow instructions)

3. RLHF / DPO      → Align with human preferences
                      (safety, helpfulness, harmlessness)
```

---

## Model Landscape

| Model | Provider | Parameters | Context Window | Open/Closed |
|-------|----------|-----------|----------------|-------------|
| GPT-4o | OpenAI | Undisclosed | 128K | Closed (API) |
| Claude | Anthropic | Undisclosed | 200K | Closed (API) |
| Gemini | Google | Undisclosed | 1M+ | Closed (API) |
| LLaMA 3 | Meta | 8B, 70B, 405B | 128K | Open weights |
| Mistral | Mistral AI | 7B, 8x7B, Large | 32K-128K | Open / API |
| Phi-3 | Microsoft | 3.8B, 14B | 128K | Open weights |

**Key trade-offs:** Larger models are smarter but slower and more expensive. Smaller models can be fine-tuned for specific tasks and run locally.

---

## Using LLM APIs

### OpenAI (Chat Completions)

```python
from openai import OpenAI

client = OpenAI()  # uses OPENAI_API_KEY env var

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain binary search in one sentence."}
    ],
    temperature=0.7,
    max_tokens=100,
)

print(response.choices[0].message.content)
# "Binary search repeatedly halves a sorted array to find a
#  target in O(log n) time."
```

```javascript
import OpenAI from 'openai';

const client = new OpenAI(); // uses OPENAI_API_KEY env var

const response = await client.chat.completions.create({
  model: 'gpt-4o',
  messages: [
    { role: 'system', content: 'You are a helpful assistant.' },
    { role: 'user', content: 'Explain binary search in one sentence.' }
  ],
  temperature: 0.7,
  max_tokens: 100,
});

console.log(response.choices[0].message.content);
```

```csharp
using Azure.AI.OpenAI;

var client = new OpenAIClient(new Uri(endpoint), new AzureKeyCredential(key));

var options = new ChatCompletionsOptions
{
    DeploymentName = "gpt-4o",
    Messages =
    {
        new ChatRequestSystemMessage("You are a helpful assistant."),
        new ChatRequestUserMessage("Explain binary search in one sentence.")
    },
    Temperature = 0.7f,
    MaxTokens = 100,
};

var response = await client.GetChatCompletionsAsync(options);
Console.WriteLine(response.Value.Choices[0].Message.Content);
```

---

## Fine-Tuning

Fine-tuning adapts a pre-trained model to a specific task using your own data.

### When to Fine-Tune vs Prompt

| Approach | Best For | Cost | Effort |
|----------|----------|------|--------|
| **Prompt engineering** | Most tasks, prototyping | Per-token API cost | Low |
| **Few-shot prompting** | When examples help format/style | Higher token cost | Low |
| **RAG** | Adding knowledge the model doesn't have | Moderate | Medium |
| **Fine-tuning** | Consistent style/format, domain specialization | Training + inference | High |
| **Pre-training from scratch** | New languages, highly specialized domains | Very high | Very high |

### Fine-Tuning with OpenAI

```python
# 1. Prepare JSONL training data
# {"messages": [{"role": "system", "content": "..."}, {"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]}

# 2. Upload and create fine-tuning job
from openai import OpenAI
client = OpenAI()

file = client.files.create(file=open("training.jsonl", "rb"), purpose="fine-tune")

job = client.fine_tuning.jobs.create(
    training_file=file.id,
    model="gpt-4o-mini",
)

# 3. Use your fine-tuned model
response = client.chat.completions.create(
    model="ft:gpt-4o-mini:my-org::abc123",  # your fine-tuned model ID
    messages=[{"role": "user", "content": "..."}]
)
```

---

## Key Parameters

| Parameter | Purpose | Typical Values |
|-----------|---------|---------------|
| `temperature` | Randomness (0 = deterministic, 2 = very random) | 0-0.7 for factual, 0.7-1.2 for creative |
| `max_tokens` | Maximum output length | Task-dependent |
| `top_p` | Nucleus sampling (alternative to temperature) | 0.9-1.0 |
| `frequency_penalty` | Penalize repeated tokens | 0-1 |
| `presence_penalty` | Encourage topic diversity | 0-1 |
| `stop` | Stop sequences to end generation | `["\n", "END"]` |

---

## Cost Optimization

- **Use the smallest model that works** — gpt-4o-mini is 30x cheaper than gpt-4o
- **Cache responses** — Same prompt = same response (with temperature 0)
- **Reduce input tokens** — Trim system prompts, summarize context
- **Batch requests** — Use batch API for non-real-time tasks (50% discount)
- **Stream responses** — Better UX without reducing cost
- **Fine-tune smaller models** — Replace expensive large model + long prompt with cheap fine-tuned small model
