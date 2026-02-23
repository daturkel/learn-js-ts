# Exercise: Vercel AI SDK Chat UI

Build a streaming chat UI with Next.js and the Vercel AI SDK.

## Setup

Create a new Next.js project (or use your existing one):

```bash
npx create-next-app@latest ai-chat-app --typescript --app --tailwind --eslint --no-src-dir --import-alias "@/*"
cd ai-chat-app
npm install ai @ai-sdk/anthropic @ai-sdk/openai
```

Set your API key:
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

Or create a `.env.local` file (Next.js auto-loads this):
```
ANTHROPIC_API_KEY=sk-ant-...
```

## Tasks

### 1. Basic chat API route

Create `app/api/chat/route.ts`:

- Accept POST requests with a `messages` array
- Use `streamText` from the Vercel AI SDK with Claude
- Return a streaming response

### 2. Chat UI with `useChat`

Create `app/chat/page.tsx`:

- Use the `useChat` hook from `@ai-sdk/react`
- Display message history (user and assistant messages)
- Input field with a submit button
- Show a loading indicator while streaming
- Auto-scroll to the bottom as new tokens arrive

### 3. System prompt selector

Add a dropdown to choose different system prompts:
- "ML Tutor" — explains ML concepts with code examples
- "Code Reviewer" — reviews code snippets for improvements
- "Paper Summarizer" — summarizes research papers concisely

Pass the selected system prompt to the API route.

### 4. Model switcher (stretch)

Add a toggle to switch between Claude and GPT-4o:
- Pass the model choice to the API route
- Display which model is responding

### 5. Tool use in chat (stretch)

Add a tool to the chat that can run calculations:
- Define a `calculate` tool in the API route
- When the user asks a math question, the model calls the tool
- Display the tool call and result in the chat

<details>
<summary>Hint: useChat hook</summary>

```typescript
"use client";
import { useChat } from "@ai-sdk/react";

export default function ChatPage() {
  const {
    messages,        // Array of { id, role, content }
    input,           // Current input value
    handleInputChange,  // onChange handler for input
    handleSubmit,    // onSubmit handler for form
    isLoading,       // true while streaming
    error,           // Error object if request failed
    setMessages,     // Manually set messages (for clearing)
  } = useChat();

  // ...
}
```

To pass extra data (like system prompt) to the API route:

```typescript
const { messages, ... } = useChat({
  body: {
    systemPrompt: selectedPrompt,
  },
});
```

Then in the API route:

```typescript
export async function POST(req: Request) {
  const { messages, systemPrompt } = await req.json();
  // ...
}
```

</details>

<details>
<summary>Solution</summary>

**`app/api/chat/route.ts`**

```typescript
import { streamText } from "ai";
import { anthropic } from "@ai-sdk/anthropic";

export async function POST(req: Request) {
  const { messages, systemPrompt } = await req.json();

  const result = streamText({
    model: anthropic("claude-sonnet-4-20250514"),
    system: systemPrompt || "You are a helpful AI assistant.",
    messages,
  });

  return result.toDataStreamResponse();
}
```

**`app/chat/page.tsx`**

```typescript
"use client";

import { useChat } from "@ai-sdk/react";
import { useEffect, useRef, useState } from "react";

const SYSTEM_PROMPTS: Record<string, string> = {
  "ML Tutor":
    "You are an ML tutor. Explain concepts clearly using Python code examples. Use analogies when helpful.",
  "Code Reviewer":
    "You are a code reviewer. When shown code, provide constructive feedback on correctness, style, and potential improvements. Be specific.",
  "Paper Summarizer":
    "You are a research paper summarizer. When given a paper title or abstract, provide a concise summary covering: key contribution, method, results, and limitations.",
};

export default function ChatPage() {
  const [systemPrompt, setSystemPrompt] = useState("ML Tutor");
  const messagesEndRef = useRef<HTMLDivElement>(null);

  const { messages, input, handleInputChange, handleSubmit, isLoading, setMessages } =
    useChat({
      body: {
        systemPrompt: SYSTEM_PROMPTS[systemPrompt],
      },
    });

  // Auto-scroll to bottom
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages]);

  return (
    <div style={{ maxWidth: "700px", margin: "0 auto", padding: "1rem", height: "100vh", display: "flex", flexDirection: "column" }}>
      {/* Header */}
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: "1rem" }}>
        <h1 style={{ margin: 0 }}>AI Chat</h1>
        <div style={{ display: "flex", gap: "0.5rem", alignItems: "center" }}>
          <label htmlFor="prompt-select">Mode:</label>
          <select
            id="prompt-select"
            value={systemPrompt}
            onChange={(e) => {
              setSystemPrompt(e.target.value);
              setMessages([]);  // Clear chat when switching modes
            }}
            style={{ padding: "0.25rem" }}
          >
            {Object.keys(SYSTEM_PROMPTS).map((name) => (
              <option key={name} value={name}>{name}</option>
            ))}
          </select>
        </div>
      </div>

      {/* Messages */}
      <div style={{ flex: 1, overflowY: "auto", border: "1px solid #ccc", borderRadius: "8px", padding: "1rem", marginBottom: "1rem" }}>
        {messages.length === 0 && (
          <p style={{ color: "#999", textAlign: "center" }}>
            Start a conversation with the {systemPrompt}...
          </p>
        )}

        {messages.map((message) => (
          <div
            key={message.id}
            style={{
              marginBottom: "1rem",
              padding: "0.75rem",
              borderRadius: "8px",
              backgroundColor: message.role === "user" ? "#e8f4f8" : "#f5f5f5",
            }}
          >
            <div style={{ fontWeight: "bold", marginBottom: "0.25rem", fontSize: "0.85rem" }}>
              {message.role === "user" ? "You" : "Assistant"}
            </div>
            <div style={{ whiteSpace: "pre-wrap" }}>{message.content}</div>
          </div>
        ))}

        {isLoading && messages[messages.length - 1]?.role === "user" && (
          <div style={{ color: "#999", padding: "0.75rem" }}>Thinking...</div>
        )}

        <div ref={messagesEndRef} />
      </div>

      {/* Input */}
      <form onSubmit={handleSubmit} style={{ display: "flex", gap: "0.5rem" }}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Type a message..."
          style={{ flex: 1, padding: "0.75rem", borderRadius: "8px", border: "1px solid #ccc" }}
          disabled={isLoading}
        />
        <button
          type="submit"
          disabled={isLoading || !input.trim()}
          style={{
            padding: "0.75rem 1.5rem",
            borderRadius: "8px",
            border: "none",
            backgroundColor: isLoading ? "#ccc" : "#0070f3",
            color: "white",
            cursor: isLoading ? "not-allowed" : "pointer",
          }}
        >
          Send
        </button>
      </form>
    </div>
  );
}
```

**Update `app/layout.tsx`** to add a link to `/chat`:

```typescript
<a href="/chat">Chat</a>
```

### How it works

```
Browser (React)              Server (API Route)           Anthropic API
     │                            │                            │
     ├─ useChat sends POST ──────►│                            │
     │   { messages, systemPrompt}│                            │
     │                            ├─ streamText() ────────────►│
     │                            │                            │
     │                            │◄──── streaming tokens ─────┤
     │◄──── SSE stream ──────────┤                            │
     │   (updates messages        │                            │
     │    state automatically)    │                            │
     │                            │                            │
```

The `useChat` hook:
1. Manages message state (history of user + assistant messages)
2. Sends messages to your API route on submit
3. Reads the streaming response and updates the UI token by token
4. Handles loading states and errors

This is the pattern used by ChatGPT, Claude.ai, and most AI chat interfaces.

### Comparing to building it manually

Without the Vercel AI SDK, you'd need to:
1. Manage message state yourself (`useState`)
2. Send fetch requests on form submit
3. Read the SSE/streaming response manually (like Module 03, Exercise 04)
4. Parse streaming chunks and append to the message
5. Handle errors, loading states, abort, etc.

The `useChat` hook handles all of this in one line.

</details>

---

**Next:** [Module 11: MCP Servers →](/11-mcp-servers/README.md)
