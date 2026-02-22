# Exercise: Streaming Responses

Stream tokens from AI models to the terminal in real time.

## Setup

Use the same project from Exercise 01.

## Tasks

### 1. Basic streaming with Anthropic

Create `src/stream-basic.ts`:

- Use `client.messages.stream()` to stream a response
- Print each text token as it arrives (no newline between tokens)
- After the stream completes, print the total token usage

### 2. Event-based streaming

Create `src/stream-events.ts`:

- Use the stream's event types to handle different events:
  - `message_start` — log the model being used
  - `content_block_delta` — print text deltas
  - `message_stop` — print a summary line
- Print timing info: time to first token, total time

### 3. OpenAI streaming

Create `src/stream-openai.ts`:

- Stream a response from GPT-4o
- OpenAI uses `stream: true` instead of a separate method
- Print tokens as they arrive

### 4. Streaming with abort

Create `src/stream-abort.ts`:

- Start streaming a long response (ask for a 500-word essay)
- Cancel the stream after 5 seconds using `AbortController`
- Print how many tokens were received before cancellation

### 5. Collect and process

Create `src/stream-collect.ts`:

- Stream a response that produces structured data (ask for a JSON list of 5 ML papers)
- Collect all tokens into a string
- Parse the final collected string as JSON
- Print the parsed data

This demonstrates a common pattern: stream for UX (show progress), but also collect the full response for processing.

<details>
<summary>Hint: Stream methods</summary>

The Anthropic SDK provides multiple ways to consume streams:

```typescript
// Method 1: for-await loop over events
const stream = client.messages.stream({ ... });
for await (const event of stream) {
  if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
    process.stdout.write(event.delta.text);
  }
}

// Method 2: Event handlers
const stream = client.messages.stream({ ... });
stream.on("text", (text) => process.stdout.write(text));
const finalMessage = await stream.finalMessage();

// Method 3: Get the final message directly (no streaming UI)
const stream = client.messages.stream({ ... });
const message = await stream.finalMessage();
```

</details>

<details>
<summary>Solution</summary>

**`src/stream-basic.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const stream = client.messages.stream({
  model: "claude-sonnet-4-20250514",
  max_tokens: 512,
  messages: [
    { role: "user", content: "Explain how attention mechanisms work in transformers. Be thorough." },
  ],
});

// Print tokens as they arrive
stream.on("text", (text) => {
  process.stdout.write(text);
});

// Wait for completion and print usage
const finalMessage = await stream.finalMessage();

console.log("\n\n--- Usage ---");
console.log(`Input tokens:  ${finalMessage.usage.input_tokens}`);
console.log(`Output tokens: ${finalMessage.usage.output_tokens}`);
console.log(`Stop reason:   ${finalMessage.stop_reason}`);
```

**`src/stream-events.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const startTime = Date.now();
let firstTokenTime: number | null = null;
let tokenCount = 0;

const stream = client.messages.stream({
  model: "claude-sonnet-4-20250514",
  max_tokens: 512,
  messages: [
    { role: "user", content: "Write a short explanation of batch normalization." },
  ],
});

for await (const event of stream) {
  switch (event.type) {
    case "message_start":
      console.log(`Model: ${event.message.model}`);
      console.log(`---`);
      break;

    case "content_block_delta":
      if (event.delta.type === "text_delta") {
        if (firstTokenTime === null) {
          firstTokenTime = Date.now();
        }
        process.stdout.write(event.delta.text);
        tokenCount++;
      }
      break;

    case "message_stop":
      const totalTime = Date.now() - startTime;
      const ttft = firstTokenTime ? firstTokenTime - startTime : 0;
      console.log(`\n---`);
      console.log(`Time to first token: ${ttft}ms`);
      console.log(`Total time:          ${totalTime}ms`);
      console.log(`Text deltas:         ${tokenCount}`);
      break;
  }
}
```

**`src/stream-openai.ts`**

```typescript
import OpenAI from "openai";

const client = new OpenAI();

const stream = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [
    { role: "user", content: "Explain the vanishing gradient problem in 3 paragraphs." },
  ],
  stream: true,
});

for await (const chunk of stream) {
  const text = chunk.choices[0]?.delta?.content;
  if (text) {
    process.stdout.write(text);
  }
}

console.log("\n\nDone.");
```

**`src/stream-abort.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const controller = new AbortController();
let collected = "";

// Cancel after 5 seconds
const timeout = setTimeout(() => {
  console.log("\n\n--- Cancelling after 5 seconds ---");
  controller.abort();
}, 5000);

try {
  const stream = client.messages.stream(
    {
      model: "claude-sonnet-4-20250514",
      max_tokens: 2048,
      messages: [
        { role: "user", content: "Write a 500-word essay about the history of neural networks." },
      ],
    },
    { signal: controller.signal },
  );

  for await (const event of stream) {
    if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
      process.stdout.write(event.delta.text);
      collected += event.delta.text;
    }
  }
} catch (error) {
  if (error instanceof Error && error.name === "AbortError") {
    console.log(`\nReceived ${collected.length} characters before abort.`);
    console.log(`Word count: ~${collected.split(/\s+/).length}`);
  } else {
    throw error;
  }
} finally {
  clearTimeout(timeout);
}
```

**`src/stream-collect.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const stream = client.messages.stream({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [
    {
      role: "user",
      content: `List 5 influential ML papers as a JSON array. Each object should have:
- "title": paper title
- "year": publication year
- "key_contribution": one sentence

Return ONLY the JSON array, no markdown formatting.`,
    },
  ],
});

// Show streaming progress
let collected = "";
process.stdout.write("Streaming: ");
stream.on("text", (text) => {
  collected += text;
  process.stdout.write(".");  // progress indicator
});

await stream.finalMessage();
console.log(" done!\n");

// Parse the collected response
try {
  const papers = JSON.parse(collected);
  console.log(`Received ${papers.length} papers:\n`);
  for (const paper of papers) {
    console.log(`  ${paper.year} — ${paper.title}`);
    console.log(`         ${paper.key_contribution}\n`);
  }
} catch {
  console.error("Failed to parse JSON:");
  console.error(collected);
}
```

### Why streaming matters

In a CLI, streaming gives users immediate feedback instead of a long wait. In a web UI (Module 09 + Exercise 04 of this module), streaming makes chat feel responsive.

The pattern is the same one from [Module 03, Exercise 04](../../03-async-patterns/exercises/04-streaming.md):

```
Server produces tokens → Stream over network → Client renders incrementally
```

Python equivalent (Anthropic):
```python
with client.messages.stream(...) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

TypeScript:
```typescript
const stream = client.messages.stream({ ... });
stream.on("text", (text) => process.stdout.write(text));
await stream.finalMessage();
```

</details>
