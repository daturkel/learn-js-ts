# Exercise: Tool Use (Function Calling)

Give Claude tools it can call, handle the tool call loop, and build a simple agent.

## Setup

Use the same project from Exercise 01.

## Background

Tool use follows a loop:

1. You send a message with available tools
2. The model responds with a `tool_use` content block (instead of or alongside text)
3. Your code executes the tool and gets a result
4. You send the result back as a `tool_result` message
5. The model continues (possibly calling more tools or giving a final answer)

This is the same pattern regardless of SDK — Anthropic, OpenAI, or Vercel AI SDK.

## Requirements

### 1. Single tool call

Create `src/tool-single.ts`:

Define a `calculate` tool that can do basic math:
- Parameters: `expression` (string like "2 + 3" or "sqrt(16)")
- Your code evaluates it and returns the result
- Send a message like "What is 47 * 23 + 15?"

Handle the tool_use → tool_result → final response loop.

### 2. Multiple tools

Create `src/tool-multi.ts`:

Define these tools for an ML experiment helper:
- `list_experiments` — returns a list of experiment names and IDs
- `get_experiment` — takes an `id`, returns experiment details
- `compare_experiments` — takes two `id`s, returns a comparison

Hardcode the data (no database needed). Send a prompt like "Compare my best two experiments" and let the model figure out which tools to call and in what order.

### 3. Tool call loop

Create `src/tool-loop.ts`:

Implement the full agentic loop:

```
while model wants to use tools:
    execute each tool
    send results back
    get next response
print final text response
```

The model might need multiple rounds of tool calls to answer a question. Your code needs to keep looping until the model gives a final text response (`stop_reason === "end_stop"`).

### 4. Weather agent (stretch)

Create `src/weather-agent.ts`:

Build a simple agent with these tools:
- `get_weather` — takes a city, returns fake weather data
- `get_forecast` — takes a city and number of days, returns fake forecast
- `convert_temperature` — converts between Celsius and Fahrenheit

Have a conversation: "What's the weather like in Tokyo? Should I bring a jacket?"

<details>
<summary>Hint: Tool use response structure</summary>

When Claude wants to call a tool, the response looks like:

```typescript
// response.stop_reason === "tool_use"
// response.content might contain both text and tool_use blocks:
[
  { type: "text", text: "Let me look that up." },
  { type: "tool_use", id: "toolu_abc123", name: "get_weather", input: { city: "Tokyo" } }
]
```

You send the result back like this:

```typescript
messages.push(
  { role: "assistant", content: response.content },  // include the full assistant response
  {
    role: "user",
    content: [
      {
        type: "tool_result",
        tool_use_id: "toolu_abc123",  // must match the tool_use id
        content: JSON.stringify({ temperature: 22, condition: "sunny" }),
      },
    ],
  },
);
```

</details>

<details>
<summary>Solution</summary>

**`src/tool-single.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const tools: Anthropic.Messages.Tool[] = [
  {
    name: "calculate",
    description: "Evaluate a mathematical expression. Supports basic arithmetic (+, -, *, /, **) and functions like sqrt, abs, round.",
    input_schema: {
      type: "object" as const,
      properties: {
        expression: {
          type: "string",
          description: "The math expression to evaluate, e.g. '47 * 23 + 15' or 'Math.sqrt(16)'",
        },
      },
      required: ["expression"],
    },
  },
];

function executeCalculate(expression: string): string {
  try {
    // Replace common math notation with JS equivalents
    const jsExpr = expression
      .replace(/sqrt\(/g, "Math.sqrt(")
      .replace(/abs\(/g, "Math.abs(")
      .replace(/round\(/g, "Math.round(")
      .replace(/\^/g, "**");

    // Evaluate (safe enough for a learning exercise — don't do this in production)
    const result = new Function(`return ${jsExpr}`)();
    return String(result);
  } catch {
    return `Error evaluating: ${expression}`;
  }
}

// Initial request
const messages: Anthropic.Messages.MessageParam[] = [
  { role: "user", content: "What is 47 * 23 + 15? Also, what is the square root of 1764?" },
];

console.log("User:", messages[0].content);
console.log();

let response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  tools,
  messages,
});

// Tool use loop
while (response.stop_reason === "tool_use") {
  // Find tool_use blocks
  const toolUseBlocks = response.content.filter(
    (block): block is Anthropic.Messages.ToolUseBlock => block.type === "tool_use"
  );

  // Execute each tool
  const toolResults: Anthropic.Messages.ToolResultBlockParam[] = toolUseBlocks.map((block) => {
    console.log(`Tool call: ${block.name}(${JSON.stringify(block.input)})`);
    const input = block.input as { expression: string };
    const result = executeCalculate(input.expression);
    console.log(`Result: ${result}`);
    return {
      type: "tool_result" as const,
      tool_use_id: block.id,
      content: result,
    };
  });

  // Send results back
  messages.push(
    { role: "assistant", content: response.content },
    { role: "user", content: toolResults },
  );

  response = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    tools,
    messages,
  });
}

// Print final response
console.log();
for (const block of response.content) {
  if (block.type === "text") {
    console.log("Claude:", block.text);
  }
}
```

**`src/tool-multi.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

// Mock data
const experiments = [
  { id: "exp-001", name: "BERT Sentiment", model: "bert-base", accuracy: 0.92, f1: 0.90, epochs: 3, status: "completed" },
  { id: "exp-002", name: "GPT-2 Generation", model: "gpt2", accuracy: 0.78, f1: 0.75, epochs: 5, status: "completed" },
  { id: "exp-003", name: "T5 Summary", model: "t5-small", accuracy: 0.87, f1: 0.85, epochs: 4, status: "completed" },
  { id: "exp-004", name: "DistilBERT NER", model: "distilbert", accuracy: 0.83, f1: 0.81, epochs: 6, status: "completed" },
];

// Tool definitions
const tools: Anthropic.Messages.Tool[] = [
  {
    name: "list_experiments",
    description: "List all ML experiments with their IDs, names, and status",
    input_schema: {
      type: "object" as const,
      properties: {},
      required: [],
    },
  },
  {
    name: "get_experiment",
    description: "Get detailed information about a specific experiment by ID",
    input_schema: {
      type: "object" as const,
      properties: {
        id: { type: "string", description: "Experiment ID (e.g. exp-001)" },
      },
      required: ["id"],
    },
  },
  {
    name: "compare_experiments",
    description: "Compare two experiments side by side on all metrics",
    input_schema: {
      type: "object" as const,
      properties: {
        id1: { type: "string", description: "First experiment ID" },
        id2: { type: "string", description: "Second experiment ID" },
      },
      required: ["id1", "id2"],
    },
  },
];

// Tool implementations
function executeTool(name: string, input: Record<string, string>): string {
  switch (name) {
    case "list_experiments":
      return JSON.stringify(experiments.map(e => ({ id: e.id, name: e.name, status: e.status, accuracy: e.accuracy })));

    case "get_experiment": {
      const exp = experiments.find(e => e.id === input.id);
      return exp ? JSON.stringify(exp) : JSON.stringify({ error: `Experiment ${input.id} not found` });
    }

    case "compare_experiments": {
      const e1 = experiments.find(e => e.id === input.id1);
      const e2 = experiments.find(e => e.id === input.id2);
      if (!e1 || !e2) return JSON.stringify({ error: "One or both experiments not found" });
      return JSON.stringify({
        experiment1: e1,
        experiment2: e2,
        comparison: {
          accuracy_diff: +(e1.accuracy - e2.accuracy).toFixed(4),
          f1_diff: +(e1.f1 - e2.f1).toFixed(4),
          better_accuracy: e1.accuracy > e2.accuracy ? e1.name : e2.name,
          better_f1: e1.f1 > e2.f1 ? e1.name : e2.name,
        },
      });
    }

    default:
      return JSON.stringify({ error: `Unknown tool: ${name}` });
  }
}

// Agentic loop
const messages: Anthropic.Messages.MessageParam[] = [
  { role: "user", content: "Compare my best two experiments by accuracy. Which one should I deploy?" },
];

console.log("User:", messages[0].content);
console.log();

let response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  tools,
  messages,
});

let round = 0;
while (response.stop_reason === "tool_use") {
  round++;
  console.log(`--- Round ${round} ---`);

  const toolUseBlocks = response.content.filter(
    (block): block is Anthropic.Messages.ToolUseBlock => block.type === "tool_use"
  );

  // Print any text the model included
  for (const block of response.content) {
    if (block.type === "text" && block.text.trim()) {
      console.log("Claude:", block.text);
    }
  }

  const toolResults: Anthropic.Messages.ToolResultBlockParam[] = toolUseBlocks.map((block) => {
    console.log(`Calling: ${block.name}(${JSON.stringify(block.input)})`);
    const result = executeTool(block.name, block.input as Record<string, string>);
    console.log(`Result:  ${result.slice(0, 100)}${result.length > 100 ? "..." : ""}`);
    return {
      type: "tool_result" as const,
      tool_use_id: block.id,
      content: result,
    };
  });

  messages.push(
    { role: "assistant", content: response.content },
    { role: "user", content: toolResults },
  );

  response = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    tools,
    messages,
  });
}

// Final response
console.log("\n--- Final Response ---");
for (const block of response.content) {
  if (block.type === "text") {
    console.log(block.text);
  }
}
```

**`src/tool-loop.ts`**

A reusable agentic loop function:

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

type ToolExecutor = (name: string, input: Record<string, unknown>) => string | Promise<string>;

async function agentLoop(
  systemPrompt: string,
  userMessage: string,
  tools: Anthropic.Messages.Tool[],
  executeTool: ToolExecutor,
  maxRounds = 10,
): Promise<string> {
  const messages: Anthropic.Messages.MessageParam[] = [
    { role: "user", content: userMessage },
  ];

  for (let round = 0; round < maxRounds; round++) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 1024,
      system: systemPrompt,
      tools,
      messages,
    });

    // If no more tool use, return the text
    if (response.stop_reason === "end_turn" || response.stop_reason === "stop_sequence") {
      const textBlocks = response.content.filter(
        (b): b is Anthropic.Messages.TextBlock => b.type === "text"
      );
      return textBlocks.map(b => b.text).join("\n");
    }

    // Execute tools
    const toolUseBlocks = response.content.filter(
      (b): b is Anthropic.Messages.ToolUseBlock => b.type === "tool_use"
    );

    if (toolUseBlocks.length === 0) {
      // No tools and not end_turn — shouldn't happen, but handle it
      const textBlocks = response.content.filter(
        (b): b is Anthropic.Messages.TextBlock => b.type === "text"
      );
      return textBlocks.map(b => b.text).join("\n");
    }

    const toolResults: Anthropic.Messages.ToolResultBlockParam[] = [];
    for (const block of toolUseBlocks) {
      console.log(`  [tool] ${block.name}(${JSON.stringify(block.input)})`);
      const result = await executeTool(block.name, block.input as Record<string, unknown>);
      toolResults.push({
        type: "tool_result",
        tool_use_id: block.id,
        content: result,
      });
    }

    messages.push(
      { role: "assistant", content: response.content },
      { role: "user", content: toolResults },
    );
  }

  return "(max rounds exceeded)";
}

// Example usage
const tools: Anthropic.Messages.Tool[] = [
  {
    name: "get_date",
    description: "Get the current date and time",
    input_schema: { type: "object" as const, properties: {}, required: [] },
  },
  {
    name: "calculate",
    description: "Evaluate a math expression",
    input_schema: {
      type: "object" as const,
      properties: {
        expression: { type: "string" },
      },
      required: ["expression"],
    },
  },
];

function executeTool(name: string, input: Record<string, unknown>): string {
  if (name === "get_date") return new Date().toISOString();
  if (name === "calculate") {
    try {
      return String(new Function(`return ${input.expression}`)());
    } catch {
      return "Error evaluating expression";
    }
  }
  return "Unknown tool";
}

const result = await agentLoop(
  "You are a helpful assistant with access to tools.",
  "What day of the week is it? Also, what is 2^10?",
  tools,
  executeTool,
);

console.log("\nFinal answer:", result);
```

### Key takeaways

The tool use pattern is the same across all SDKs:

```
User message → Model (may request tool) → Execute tool → Send result → Model continues
                    ↑___________________________________________|
```

This is the foundation of "agents" — models that can take actions in the real world by calling tools in a loop until they have enough information to answer.

In Python, the pattern is identical:
```python
while response.stop_reason == "tool_use":
    # execute tools
    # send results back
    response = client.messages.create(...)
```

</details>
