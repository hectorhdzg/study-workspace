# Prompt Engineering

## Why Prompt Engineering Matters

The same model can produce wildly different results depending on how you ask. Prompt engineering is the skill of structuring inputs to get reliable, high-quality outputs.

---

## Core Techniques

### 1. System Prompts

Set the persona, rules, and output format.

```
System: You are a senior software engineer. Answer concisely with code examples.
        Always include time and space complexity. Use TypeScript.
```

### 2. Few-Shot Prompting

Provide examples of the desired input → output format.

```
Classify the sentiment of each review.

Review: "The food was amazing and the service was fast."
Sentiment: Positive

Review: "Waited 45 minutes and the order was wrong."
Sentiment: Negative

Review: "The new interface is confusing but loads faster now."
Sentiment:
```

### 3. Chain-of-Thought (CoT)

Ask the model to reason step-by-step before answering.

```
Q: A store has 15 shirts. 8 are blue, 5 are red, and the rest are green.
   What fraction of the shirts are green?

Think step by step:
1. Total shirts = 15
2. Blue + Red = 8 + 5 = 13
3. Green = 15 - 13 = 2
4. Fraction = 2/15

Answer: 2/15
```

**Why it works:** Forces the model to decompose the problem, reducing errors on multi-step reasoning tasks.

### 4. Structured Output

Request specific formats (JSON, XML, Markdown) for parseable responses.

```python
from openai import OpenAI

client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{
        "role": "user",
        "content": "Extract entities from: 'John Smith works at Google in NYC.'"
    }],
    response_format={"type": "json_object"},  # enforces valid JSON
)

# {"name": "John Smith", "company": "Google", "location": "NYC"}
```

```csharp
var options = new ChatCompletionsOptions
{
    DeploymentName = "gpt-4o",
    Messages = { new ChatRequestUserMessage(
        "Extract entities from: 'John Smith works at Google in NYC.' Return as JSON.") },
    ResponseFormat = ChatCompletionsResponseFormat.JsonObject,
};
```

### 5. Self-Consistency

Run the same prompt multiple times with temperature > 0, then take the majority answer. Improves accuracy on reasoning tasks.

### 6. ReAct (Reasoning + Acting)

Combine reasoning with tool use in an interleaved loop:

```
Thought: I need to find the current stock price of AAPL.
Action: search("AAPL stock price today")
Observation: AAPL is trading at $198.50.
Thought: Now I can answer the user's question.
Answer: Apple (AAPL) is currently trading at $198.50.
```

---

## Prompt Patterns for Interviews

### Code Review Prompt

```
Review the following code for bugs, performance issues, and best practices.
For each issue found:
1. State the problem
2. Explain why it's a problem
3. Provide the corrected code

Code:
{paste code here}
```

### System Design Brainstorm

```
I'm designing a {system}. Help me think through:
1. Requirements (functional + non-functional)
2. API design
3. Data model
4. High-level architecture
5. Key trade-offs

Start with clarifying questions before proposing a design.
```

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| Vague instructions | Model guesses what you want | Be specific about format, length, style |
| No examples | Inconsistent output format | Add 2-3 few-shot examples |
| Too much context | Noise drowns out the task | Put the most important info last (recency bias) |
| Asking "do you understand?" | Model always says yes | Instead, ask it to restate the task |
| Long chain of conversation | Context window fills up, quality degrades | Summarize and reset periodically |

---

## Token Optimization

```python
# Bad: 150 tokens of system prompt
"You are a highly knowledgeable, expert-level assistant that specializes in..."

# Good: 30 tokens, same effect
"You are an expert Python developer. Be concise."
```

Keep system prompts tight — every token costs money and latency.
