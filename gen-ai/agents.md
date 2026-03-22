# AI Agents & Tool Use

## What Are AI Agents?

An **AI agent** is an LLM that can take actions — calling tools, querying APIs, reading files, and executing code — in a loop until a task is complete.

```
User Task
    ↓
┌─────────────────────────┐
│  LLM reasons about task │ ← Think
│  Decides which tool     │ ← Act
│  Executes tool          │
│  Observes result        │ ← Observe
│  Repeat until done      │
└─────────────────────────┘
    ↓
Final Answer
```

---

## Function Calling (Tool Use)

Modern LLMs can select and call functions you define. The model chooses which function to call and generates the arguments as structured JSON.

### OpenAI Function Calling

```python
from openai import OpenAI
import json

client = OpenAI()

# Define available tools
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string", "description": "City name"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
                },
                "required": ["location"]
            }
        }
    }
]

# LLM decides to call the function
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
    tools=tools,
)

# Extract the function call
tool_call = response.choices[0].message.tool_calls[0]
print(tool_call.function.name)       # "get_weather"
print(tool_call.function.arguments)  # '{"location": "Tokyo", "unit": "celsius"}'

# Execute the function and return result to the model
result = get_weather("Tokyo", "celsius")  # your function

follow_up = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "What's the weather in Tokyo?"},
        response.choices[0].message,   # assistant's tool call
        {"role": "tool", "tool_call_id": tool_call.id, "content": json.dumps(result)}
    ],
    tools=tools,
)
print(follow_up.choices[0].message.content)
# "The current weather in Tokyo is 22°C and sunny."
```

```javascript
const tools = [{
  type: 'function',
  function: {
    name: 'get_weather',
    description: 'Get the current weather for a location',
    parameters: {
      type: 'object',
      properties: {
        location: { type: 'string', description: 'City name' },
        unit: { type: 'string', enum: ['celsius', 'fahrenheit'] }
      },
      required: ['location']
    }
  }
}];

const response = await client.chat.completions.create({
  model: 'gpt-4o',
  messages: [{ role: 'user', content: "What's the weather in Tokyo?" }],
  tools,
});

const toolCall = response.choices[0].message.tool_calls[0];
console.log(toolCall.function.name);       // "get_weather"
console.log(toolCall.function.arguments);  // '{"location": "Tokyo"}'
```

---

## Agentic Patterns

### 1. ReAct (Reasoning + Acting)

The model alternates between thinking and acting:

```
Thought: I need to find the user's order status.
Action: query_database(order_id="ORD-1234")
Observation: {"status": "shipped", "tracking": "1Z999..."}
Thought: The order has shipped. I should provide the tracking number.
Answer: Your order ORD-1234 has shipped! Tracking: 1Z999...
```

### 2. Plan-and-Execute

Break the task into steps first, then execute them:

```
Plan:
1. Search for recent papers on multi-agent systems
2. Summarize the top 3 results
3. Compare their approaches
4. Write a summary table

Executing step 1...
Executing step 2...
...
```

### 3. Multi-Agent

Multiple specialized agents collaborate:

```
[Orchestrator Agent]
    ├── [Research Agent]   → searches and summarizes info
    ├── [Code Agent]       → writes and tests code
    └── [Review Agent]     → checks quality and correctness
```

### 4. Reflection / Self-Critique

Agent reviews its own output and iterates:

```
Generate → Critique → Revise → Critique → Final Answer
```

---

## Agent Frameworks

| Framework | Language | Strengths |
|-----------|----------|-----------|
| **LangChain** | Python, JS | Largest ecosystem, many integrations |
| **LlamaIndex** | Python | Best for RAG-focused agents |
| **Semantic Kernel** | C#, Python, Java | Microsoft ecosystem, enterprise |
| **AutoGen** | Python | Multi-agent conversations |
| **CrewAI** | Python | Role-based multi-agent |

### Semantic Kernel (C#)

```csharp
using Microsoft.SemanticKernel;

var builder = Kernel.CreateBuilder();
builder.AddAzureOpenAIChatCompletion("gpt-4o", endpoint, apiKey);

// Register a native function as a tool
builder.Plugins.AddFromType<WeatherPlugin>();

var kernel = builder.Build();

// Agent automatically uses tools when needed
var result = await kernel.InvokePromptAsync(
    "What's the weather in Tokyo? Should I bring an umbrella?");
Console.WriteLine(result);
```

---

## Agent Safety & Guardrails

| Risk | Mitigation |
|------|-----------|
| **Infinite loops** | Max iterations / timeout |
| **Wrong tool calls** | Input validation, sandboxing |
| **Data leakage** | Restrict tool access by user role |
| **Prompt injection** | Separate system/user context, input sanitization |
| **Excessive cost** | Token budgets, rate limiting |
| **Unintended actions** | Human-in-the-loop approval for destructive operations |
