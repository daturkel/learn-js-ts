# Exercise: Chat Completion

Make your first API calls to Claude and GPT from TypeScript.

## Setup

```bash
mkdir ai-chat && cd ai-chat
npm init -y
npm install @anthropic-ai/sdk openai
npm install -D typescript tsx
```

Add to `package.json`: `"type": "module"`

Set your API keys:
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."   # optional — skip OpenAI tasks if you don't have a key
```

## Requirements

### 1. Basic Claude call

Create `src/basic.ts`:

- Create an Anthropic client
- Send a message asking Claude to explain what a loss function is, in 2-3 sentences
- Print the response text
- Also print the usage stats (`input_tokens`, `output_tokens`)

Run with `npx tsx src/basic.ts`.

### 2. System prompt

Create `src/tutor.ts`:

- Use a system prompt: "You are an ML tutor. Explain concepts using Python code examples. Keep answers under 200 words."
- Send a user message: "What is dropout regularization?"
- Print the response

### 3. Multi-turn conversation

Create `src/conversation.ts`:

- Build a multi-turn conversation array
- First ask: "What is a learning rate?"
- Include a mock assistant response (or use the actual response from a first call)
- Follow up with: "What happens if it's too high?"
- Print the final response

### 4. Compare providers

Create `src/compare.ts`:

- Send the same prompt to both Claude and GPT-4o
- Print both responses side by side
- Compare the usage/token counts
- Time both calls using `Date.now()` or `performance.now()`

### 5. Error handling

Create `src/errors.ts`:

- Handle these error cases:
  - Invalid API key (catch the authentication error)
  - Rate limiting (429 status)
  - Invalid model name
- Print a user-friendly message for each

<details>
<summary>Hint: Response structure</summary>

Anthropic response:
```typescript
// message.content is an array — usually one text block
const text = message.content[0].type === "text" ? message.content[0].text : "";

// Usage
console.log(`Input tokens: ${message.usage.input_tokens}`);
console.log(`Output tokens: ${message.usage.output_tokens}`);
```

OpenAI response:
```typescript
// Single string (or null)
const text = completion.choices[0].message.content ?? "";

// Usage
console.log(`Input tokens: ${completion.usage?.prompt_tokens}`);
console.log(`Output tokens: ${completion.usage?.completion_tokens}`);
```

</details>

<details>
<summary>Solution</summary>

**`src/basic.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const message = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 256,
  messages: [
    { role: "user", content: "Explain what a loss function is in 2-3 sentences." },
  ],
});

if (message.content[0].type === "text") {
  console.log(message.content[0].text);
}

console.log("\n--- Usage ---");
console.log(`Input tokens:  ${message.usage.input_tokens}`);
console.log(`Output tokens: ${message.usage.output_tokens}`);
console.log(`Stop reason:   ${message.stop_reason}`);
```

**`src/tutor.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const message = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  system: "You are an ML tutor. Explain concepts using Python code examples. Keep answers under 200 words.",
  messages: [
    { role: "user", content: "What is dropout regularization?" },
  ],
});

if (message.content[0].type === "text") {
  console.log(message.content[0].text);
}
```

**`src/conversation.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

// First turn
const first = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 512,
  messages: [
    { role: "user", content: "What is a learning rate? Keep it brief." },
  ],
});

const firstResponse = first.content[0].type === "text" ? first.content[0].text : "";
console.log("=== Turn 1 ===");
console.log(firstResponse);

// Second turn — include the conversation history
const second = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 512,
  messages: [
    { role: "user", content: "What is a learning rate? Keep it brief." },
    { role: "assistant", content: firstResponse },
    { role: "user", content: "What happens if it's too high?" },
  ],
});

console.log("\n=== Turn 2 ===");
if (second.content[0].type === "text") {
  console.log(second.content[0].text);
}
```

**`src/compare.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";
import OpenAI from "openai";

const anthropic = new Anthropic();
const openai = new OpenAI();

const prompt = "What is the difference between supervised and unsupervised learning? Answer in 3 sentences.";

// Run both in parallel
const [claudeResult, gptResult] = await Promise.allSettled([
  (async () => {
    const start = Date.now();
    const msg = await anthropic.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 256,
      messages: [{ role: "user", content: prompt }],
    });
    const elapsed = Date.now() - start;
    const text = msg.content[0].type === "text" ? msg.content[0].text : "";
    return { text, elapsed, tokens: msg.usage };
  })(),
  (async () => {
    const start = Date.now();
    const completion = await openai.chat.completions.create({
      model: "gpt-4o",
      messages: [{ role: "user", content: prompt }],
      max_tokens: 256,
    });
    const elapsed = Date.now() - start;
    const text = completion.choices[0].message.content ?? "";
    return { text, elapsed, tokens: completion.usage };
  })(),
]);

console.log("=== Claude ===");
if (claudeResult.status === "fulfilled") {
  const r = claudeResult.value;
  console.log(r.text);
  console.log(`\nTime: ${r.elapsed}ms | Tokens: ${r.tokens.input_tokens} in, ${r.tokens.output_tokens} out`);
} else {
  console.log(`Error: ${claudeResult.reason}`);
}

console.log("\n=== GPT-4o ===");
if (gptResult.status === "fulfilled") {
  const r = gptResult.value;
  console.log(r.text);
  console.log(`\nTime: ${r.elapsed}ms | Tokens: ${r.tokens?.prompt_tokens} in, ${r.tokens?.completion_tokens} out`);
} else {
  console.log(`Error: ${gptResult.reason}`);
}
```

**`src/errors.ts`**

```typescript
import Anthropic from "@anthropic-ai/sdk";

// Test with a bad key
async function testBadKey() {
  const client = new Anthropic({ apiKey: "sk-ant-invalid" });
  try {
    await client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 64,
      messages: [{ role: "user", content: "Hello" }],
    });
  } catch (error) {
    if (error instanceof Anthropic.AuthenticationError) {
      console.log("Auth error (expected):", error.message);
    } else if (error instanceof Anthropic.RateLimitError) {
      console.log("Rate limited — wait and retry");
    } else if (error instanceof Anthropic.BadRequestError) {
      console.log("Bad request:", error.message);
    } else {
      console.log("Unknown error:", error);
    }
  }
}

// Test with a bad model
async function testBadModel() {
  const client = new Anthropic();
  try {
    await client.messages.create({
      model: "claude-nonexistent-model",
      max_tokens: 64,
      messages: [{ role: "user", content: "Hello" }],
    });
  } catch (error) {
    if (error instanceof Anthropic.NotFoundError) {
      console.log("Model not found (expected):", error.message);
    } else if (error instanceof Error) {
      console.log("Error:", error.message);
    }
  }
}

await testBadKey();
await testBadModel();
```

### Python comparison

The structure is nearly identical to the Python SDK:

```python
# Python
import anthropic
client = anthropic.Anthropic()
message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
)
print(message.content[0].text)
```

```typescript
// TypeScript
import Anthropic from "@anthropic-ai/sdk";
const client = new Anthropic();
const message = await client.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello" }],
});
console.log(message.content[0].text);
```

Main differences:
- `await` required in TS (async by default)
- `new Anthropic()` instead of `anthropic.Anthropic()`
- Named imports (`import Anthropic from "..."`)

</details>
