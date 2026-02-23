# Module 10: AI SDKs

Use the Anthropic, OpenAI, and Vercel AI SDKs to build AI-powered applications in TypeScript.

**Time:** 2-3 sessions

## Why AI SDKs in TypeScript?

Most AI-powered web apps (ChatGPT, Claude.ai, Cursor, v0) are built with TypeScript. The JS/TS ecosystem has first-class support for:

- Streaming responses (built on web streams — you practiced this in [Module 03](../03-async-patterns/README.md))
- Real-time UIs (React state updates as tokens arrive)
- Edge deployment (run inference calls from Cloudflare Workers, Vercel Edge Functions)
- Tool use / function calling (typed tool definitions)

If you've used the Python `anthropic` or `openai` packages, the TypeScript versions are nearly identical in structure.

## Anthropic SDK

### Installation

```bash
npm install @anthropic-ai/sdk
```

### Basic Chat Completion

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();  // reads ANTHROPIC_API_KEY from env

const message = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "Explain gradient descent in one paragraph." }
  ],
});

// message.content is an array of content blocks
console.log(message.content[0].text);
```

Python equivalent:
```python
import anthropic

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env

message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Explain gradient descent in one paragraph."}
    ],
)

print(message.content[0].text)
```

Almost identical. The main difference: TypeScript uses `await` (the call is async), while the Python SDK provides both sync and async clients.

### Streaming

```typescript
const stream = client.messages.stream({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Write a haiku about machine learning." }],
});

for await (const event of stream) {
  if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
    process.stdout.write(event.delta.text);
  }
}
```

This uses the `for await...of` syntax you learned in [Module 03](../03-async-patterns/README.md) — async iterables are the foundation of streaming in JS/TS.

### System Prompts and Multi-Turn

```typescript
const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  system: "You are a helpful ML tutor. Keep answers concise and use code examples.",
  messages: [
    { role: "user", content: "What is a transformer?" },
    { role: "assistant", content: "A transformer is a neural network architecture..." },
    { role: "user", content: "How does self-attention work?" },
  ],
});
```

### Tool Use (Function Calling)

Tools let the model call functions you define. The model doesn't execute code — it returns structured JSON saying "I want to call this function with these arguments," and your code executes it.

```typescript
const response = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  tools: [
    {
      name: "get_weather",
      description: "Get the current weather for a location",
      input_schema: {
        type: "object" as const,
        properties: {
          location: { type: "string", description: "City name" },
          unit: { type: "string", enum: ["celsius", "fahrenheit"] },
        },
        required: ["location"],
      },
    },
  ],
  messages: [{ role: "user", content: "What's the weather in Tokyo?" }],
});

// Check if the model wants to use a tool
for (const block of response.content) {
  if (block.type === "tool_use") {
    console.log(`Tool: ${block.name}`);
    console.log(`Input: ${JSON.stringify(block.input)}`);
    // Your code would execute the tool and send results back
  }
}
```

## OpenAI SDK

### Installation

```bash
npm install openai
```

### Basic Usage

```typescript
import OpenAI from "openai";

const client = new OpenAI();  // reads OPENAI_API_KEY from env

const completion = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "system", content: "You are a helpful assistant." },
    { role: "user", content: "Explain backpropagation briefly." },
  ],
});

console.log(completion.choices[0].message.content);
```

Key differences from Anthropic:
- System prompt is a message with `role: "system"`, not a separate field
- Response is in `choices[0].message.content` (string), not `content[0].text` (array)
- Streaming uses `client.chat.completions.create({ stream: true })` instead of a separate method

### Streaming

```typescript
const stream = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "Count to 10 slowly." }],
  stream: true,
});

for await (const chunk of stream) {
  const text = chunk.choices[0]?.delta?.content;
  if (text) process.stdout.write(text);
}
```

## Vercel AI SDK

The Vercel AI SDK is a higher-level framework that abstracts away provider differences. It's especially useful for building streaming chat UIs with React/Next.js.

### Installation

```bash
npm install ai @ai-sdk/anthropic @ai-sdk/openai
```

### Unified API

```typescript
import { generateText, streamText } from "ai";
import { anthropic } from "@ai-sdk/anthropic";
import { openai } from "@ai-sdk/openai";

// Same API, different providers
const result = await generateText({
  model: anthropic("claude-sonnet-4-20250514"),
  prompt: "Explain neural networks.",
});

// Switch providers with one line change
const result2 = await generateText({
  model: openai("gpt-4o"),
  prompt: "Explain neural networks.",
});
```

### Streaming with React

The killer feature: `useChat` hook for React chat UIs.

```typescript
// app/api/chat/route.ts — server-side
import { streamText } from "ai";
import { anthropic } from "@ai-sdk/anthropic";

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = streamText({
    model: anthropic("claude-sonnet-4-20250514"),
    messages,
  });

  return result.toDataStreamResponse();
}
```

```typescript
// app/chat/page.tsx — client-side
"use client";
import { useChat } from "@ai-sdk/react";

export default function ChatPage() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat();

  return (
    <div>
      {messages.map((m) => (
        <div key={m.id}>
          <strong>{m.role}:</strong> {m.content}
        </div>
      ))}

      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} placeholder="Say something..." />
        <button type="submit" disabled={isLoading}>Send</button>
      </form>
    </div>
  );
}
```

That's a complete streaming chat UI — the `useChat` hook handles:
- Sending messages to your API route
- Streaming the response token-by-token
- Managing message history
- Loading states

### Tool Use with Vercel AI SDK

```typescript
import { generateText, tool } from "ai";
import { anthropic } from "@ai-sdk/anthropic";
import { z } from "zod";

const result = await generateText({
  model: anthropic("claude-sonnet-4-20250514"),
  prompt: "What's the weather in Paris?",
  tools: {
    getWeather: tool({
      description: "Get weather for a location",
      parameters: z.object({
        location: z.string().describe("City name"),
      }),
      execute: async ({ location }) => {
        // Your actual implementation
        return { temperature: 22, condition: "sunny" };
      },
    }),
  },
});
```

Notice: tool parameters use Zod schemas (from [Module 08](../08-web-servers-and-apis/README.md)) instead of raw JSON Schema. The SDK handles the conversion.

## SDK Comparison

| Feature | Anthropic SDK | OpenAI SDK | Vercel AI SDK |
|---------|--------------|------------|---------------|
| Provider | Anthropic only | OpenAI only | Any provider |
| Streaming | `.stream()` method | `stream: true` option | `streamText()` |
| Tool schemas | JSON Schema | JSON Schema | Zod schemas |
| React integration | Manual | Manual | `useChat` hook |
| Best for | Direct Anthropic access | Direct OpenAI access | Multi-provider apps, React UIs |

## API Keys

Set these environment variables before running exercises:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
```

Or create a `.env` file (and add `.env` to `.gitignore`).

## Further Reading

- [Anthropic TypeScript SDK](https://github.com/anthropics/anthropic-sdk-typescript)
- [OpenAI TypeScript SDK](https://github.com/openai/openai-node)
- [Vercel AI SDK Docs](https://sdk.vercel.ai/docs)
- [Anthropic Tool Use Guide](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)

## Exercises

1. [Chat Completion](./exercises/01-chat-completion.md) — Basic API calls with Anthropic and OpenAI
2. [Streaming Responses](./exercises/02-streaming-responses.md) — Stream tokens to the terminal
3. [Tool Use](./exercises/03-tool-use.md) — Give the model functions to call
4. [Vercel AI SDK](./exercises/04-vercel-ai-sdk.md) — Build a streaming chat UI with Next.js

---

**Next:** [Exercise 1: Chat Completion →](/10-ai-sdks/exercises/01-chat-completion.md)
