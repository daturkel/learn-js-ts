# Exercise: Streaming

Process data from a streaming response, chunk by chunk.

## Goal

Practice reading streaming responses — the exact pattern used to stream AI model outputs. When you call Claude or GPT with streaming enabled, tokens arrive one at a time over an HTTP stream. This exercise teaches the underlying mechanics.

## Background

In Python, streaming looks like:

```python
import httpx

async with httpx.AsyncClient() as client:
    async with client.stream("GET", url) as response:
        async for chunk in response.aiter_text():
            print(chunk, end="", flush=True)
```

In JavaScript, you use the `ReadableStream` API or async iteration.

## Setup

Create `streaming.ts`.

## Tasks

### Task 1: Read a response as a stream

Fetch a text resource and read it chunk by chunk using the low-level reader API:

```typescript
const response = await fetch("https://jsonplaceholder.typicode.com/posts");
const reader = response.body!.getReader();
const decoder = new TextDecoder();

// Read chunks in a loop
// Your code here
```

Print each chunk's size as you receive it.

### Task 2: Build a line-by-line stream reader

Many streaming APIs (including AI model APIs) send data as newline-delimited text. Build a function that reads a stream and yields complete lines:

```typescript
async function* readLines(response: Response): AsyncGenerator<string> {
  // Your code here — yield one line at a time
}
```

Test it against `https://jsonplaceholder.typicode.com/posts` (the response is a JSON array, but you can split by newlines to see the pattern).

### Task 3: Simulate an SSE (Server-Sent Events) stream

AI APIs use Server-Sent Events (SSE) format. Each event looks like:

```
data: {"token": "Hello"}

data: {"token": " world"}

data: [DONE]
```

Write a function that parses SSE-formatted text:

```typescript
function parseSSE(line: string): { token: string } | null {
  // Parse a line like 'data: {"token": "Hello"}'
  // Return null for empty lines or [DONE]
}
```

Then create a simulated stream and process it:

```typescript
// Simulate streaming tokens
async function* simulateStream(): AsyncGenerator<string> {
  const tokens = ["Hello", " from", " a", " simulated", " AI", " stream", "!"];
  for (const token of tokens) {
    await new Promise(r => setTimeout(r, 100));  // Simulate network delay
    yield `data: ${JSON.stringify({ token })}\n\n`;
  }
  yield "data: [DONE]\n\n";
}
```

Print each token as it arrives, building up the complete message.

<details>
<summary>Hint: Reading chunks</summary>

```typescript
const reader = response.body!.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  const text = decoder.decode(value, { stream: true });
  // Process text...
}
```

The `{ stream: true }` option tells the decoder that more data may follow (handles multi-byte characters split across chunks).
</details>

<details>
<summary>Hint: Line splitting with buffer</summary>

Chunks don't align with line boundaries. You need a buffer:

```typescript
let buffer = "";
// In the read loop:
buffer += text;
const lines = buffer.split("\n");
buffer = lines.pop()!;  // Last element might be incomplete
for (const line of lines) {
  yield line;
}
```
</details>

<details>
<summary>Solution</summary>

```typescript
// --- Task 1: Read chunks ---

async function readChunks() {
  console.log("=== Task 1: Reading chunks ===");
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const reader = response.body!.getReader();
  const decoder = new TextDecoder();

  let totalBytes = 0;
  let chunkCount = 0;

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const text = decoder.decode(value, { stream: true });
    totalBytes += value.byteLength;
    chunkCount++;
    console.log(`  Chunk ${chunkCount}: ${value.byteLength} bytes`);
  }

  console.log(`  Total: ${totalBytes} bytes in ${chunkCount} chunks\n`);
}

// --- Task 2: Line-by-line reader ---

async function* readLines(response: Response): AsyncGenerator<string> {
  const reader = response.body!.getReader();
  const decoder = new TextDecoder();
  let buffer = "";

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split("\n");
    buffer = lines.pop()!;  // Keep incomplete last line in buffer

    for (const line of lines) {
      if (line.trim()) {
        yield line;
      }
    }
  }

  // Flush remaining buffer
  if (buffer.trim()) {
    yield buffer;
  }
}

async function testLineReader() {
  console.log("=== Task 2: Line reader ===");
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  let lineCount = 0;

  for await (const line of readLines(response)) {
    lineCount++;
    if (lineCount <= 5) {
      console.log(`  Line ${lineCount}: ${line.slice(0, 60)}...`);
    }
  }

  console.log(`  Total lines: ${lineCount}\n`);
}

// --- Task 3: SSE parser ---

interface TokenEvent {
  token: string;
}

function parseSSE(line: string): TokenEvent | null {
  if (!line.startsWith("data: ")) return null;

  const data = line.slice(6);  // Remove "data: " prefix
  if (data === "[DONE]") return null;

  try {
    return JSON.parse(data) as TokenEvent;
  } catch {
    return null;
  }
}

async function* simulateStream(): AsyncGenerator<string> {
  const tokens = ["Hello", " from", " a", " simulated", " AI", " stream", "!"];
  for (const token of tokens) {
    await new Promise(r => setTimeout(r, 100));
    yield `data: ${JSON.stringify({ token })}\n\n`;
  }
  yield "data: [DONE]\n\n";
}

async function testSSE() {
  console.log("=== Task 3: SSE stream ===");
  let fullMessage = "";

  process.stdout.write("  ");
  for await (const chunk of simulateStream()) {
    const lines = chunk.split("\n");
    for (const line of lines) {
      const event = parseSSE(line);
      if (event) {
        fullMessage += event.token;
        process.stdout.write(event.token);
      }
    }
  }

  console.log(`\n  Complete message: "${fullMessage}"\n`);
}

// --- Run all ---

await readChunks();
await testLineReader();
await testSSE();
```

### Why this matters

When you use the Anthropic or OpenAI SDKs with streaming in Module 10, the SDK handles this SSE parsing for you. But understanding the underlying mechanics helps you:

1. Debug streaming issues
2. Build custom streaming integrations
3. Understand what `for await (const chunk of stream)` is doing under the hood

The pattern is always the same:
- Response arrives as chunks of bytes
- Decode bytes to text
- Buffer text and split on delimiters (newlines for SSE)
- Parse each complete message
- Process incrementally (print tokens as they arrive)
</details>
