# Module 03: Async Patterns

The single most important conceptual module. JavaScript's async model is the biggest paradigm shift from Python.

**Prerequisites:** [Module 01](../01-language-fundamentals/README.md), [Module 02](../02-typescript-essentials/README.md)
**Time:** 2-3 sessions

## The Event Loop

JavaScript is single-threaded. There are no threads, no multiprocessing, no GIL. All concurrency happens through the **event loop** — a single thread that processes tasks from a queue.

**Python comparison:** The closest Python equivalent is `asyncio` — single-threaded, cooperative concurrency. But in Python, asyncio is opt-in (you can still use threads, multiprocessing). In JavaScript, the event loop is the *only* execution model.

### How it works

```
┌─────────────────────────┐
│       Call Stack         │  ← Executes one function at a time
└──────────┬──────────────┘
           │ (when stack is empty)
           ▼
┌─────────────────────────┐
│     Microtask Queue     │  ← Promise callbacks (.then), queueMicrotask
└──────────┬──────────────┘
           │ (when microtasks are done)
           ▼
┌─────────────────────────┐
│      Task Queue         │  ← setTimeout callbacks, I/O callbacks
└─────────────────────────┘
```

1. Code runs on the call stack
2. Async operations (fetch, setTimeout, fs.readFile) are handed off to the runtime
3. When an async operation completes, its callback goes into a queue
4. When the call stack is empty, the event loop pulls from the queues
5. Microtasks (Promise callbacks) have priority over regular tasks

The key insight: **nothing runs in parallel**. Only one piece of your code runs at a time. "Async" means "I'll come back to this later" — not "run this on another thread."

## Callbacks (Historical)

The original async pattern. You'll see this in older code and Node.js APIs:

```typescript
// Old style — callback-based
import { readFile } from "node:fs";

readFile("config.json", "utf-8", (error, data) => {
  if (error) {
    console.error("Failed:", error);
    return;
  }
  console.log("Got:", data);
});
```

This gets ugly when you need to chain operations ("callback hell"):

```typescript
readFile("config.json", "utf-8", (err, config) => {
  if (err) return handleError(err);
  readFile(JSON.parse(config).dataPath, "utf-8", (err, data) => {
    if (err) return handleError(err);
    processData(data, (err, result) => {
      if (err) return handleError(err);
      writeFile("output.json", result, (err) => {
        // ... you get the idea
      });
    });
  });
});
```

You won't write callbacks often, but you need to recognize them. Modern code uses Promises.

## Promises

A Promise is an object that represents a value that will be available in the future. Think of it as a receipt for an async operation.

```typescript
// Creating a Promise
// `resolve` and `reject` are parameter names, not keywords.
// The Promise constructor calls your function and passes two callbacks:
//   resolve(value) — fulfills the promise with a value
//   reject(reason) — rejects the promise with an error
// You can name them anything, but `resolve`/`reject` is the convention.
const promise = new Promise<string>((resolve, reject) => {
  // Simulate async work
  setTimeout(() => {
    resolve("done!");       // Calls the resolve callback — fulfills the promise
    // reject(new Error("failed"));  // Calls the reject callback — rejects it
  }, 1000);
});

// Consuming a Promise
promise
  .then(result => console.log(result))    // "done!"
  .catch(error => console.error(error))   // Only runs if rejected
  .finally(() => console.log("cleanup")); // Always runs
```

**Python comparison:** A Promise is like an `asyncio.Future` — an object you can await to get its result. But unlike Python futures, you interact with Promises via `.then()` chains or `await`.

### Chaining

Each `.then()` returns a new Promise, enabling chains:

```typescript
fetch("https://api.example.com/data")
  .then(response => response.json())    // Returns a Promise<JSON>
  .then(data => data.results)           // Returns a Promise<Results>
  .then(results => console.log(results))
  .catch(error => console.error(error)); // Catches any error in the chain
```

## async/await

Syntactic sugar over Promises. If you've used Python's `async/await`, this will look very familiar:

```python
# Python
async def fetch_user(user_id: int) -> dict:
    response = await httpx.get(f"https://api.example.com/users/{user_id}")
    response.raise_for_status()
    return response.json()
```

```typescript
// TypeScript — nearly identical syntax
async function fetchUser(userId: number): Promise<Record<string, unknown>> {
  const response = await fetch(`https://api.example.com/users/${userId}`);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}
```

Key differences from Python:

| | Python | JavaScript |
|---|--------|-----------|
| Return type | Coroutine | Promise |
| Running | Needs `asyncio.run()` | Just call it — Node runs async natively |
| Top-level await | Only in `async def main()` | Supported in ES modules |
| Error handling | `try/except` | `try/catch` |

### Error Handling

```typescript
async function loadConfig(): Promise<Config> {
  try {
    const response = await fetch("https://api.example.com/config");
    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error("Failed to load config:", error);
    throw error;  // Re-throw to propagate
  }
}
```

### Important: fetch doesn't throw on HTTP errors

Unlike Python's `httpx` (which raises on 4xx/5xx by default), `fetch` only throws on *network* errors. You must check `response.ok`:

```typescript
const response = await fetch(url);
// response.ok is false for 4xx and 5xx — but no exception was thrown!
if (!response.ok) {
  throw new Error(`HTTP ${response.status}: ${response.statusText}`);
}
```

## Parallel Execution

### Promise.all — like asyncio.gather

Run multiple async operations concurrently:

```python
# Python
results = await asyncio.gather(
    fetch_user(1),
    fetch_user(2),
    fetch_user(3),
)
```

```typescript
// TypeScript
const results = await Promise.all([
  fetchUser(1),
  fetchUser(2),
  fetchUser(3),
]);
// results is [user1, user2, user3]
```

**Fails fast:** If any promise rejects, `Promise.all` rejects immediately (same as `asyncio.gather`).

### Promise.allSettled — handle mixed results

```typescript
const results = await Promise.allSettled([
  fetchUser(1),    // succeeds
  fetchUser(999),  // fails
  fetchUser(3),    // succeeds
]);

for (const result of results) {
  if (result.status === "fulfilled") {
    console.log("Got:", result.value);
  } else {
    console.log("Failed:", result.reason);
  }
}
```

No direct Python equivalent — `asyncio.gather(return_exceptions=True)` is the closest.

### Promise.race and Promise.any

```typescript
// First to complete (success or failure) wins
const fastest = await Promise.race([fetchFromServer1(), fetchFromServer2()]);

// First SUCCESS wins (ignores rejections until all fail)
const firstSuccess = await Promise.any([fetchFromServer1(), fetchFromServer2()]);
```

### Concurrency Limiting

Unlike Go's goroutines (unlimited) or Python's `asyncio.Semaphore`, JavaScript has no built-in concurrency limiter. You build one:

```typescript
async function pMap<T, R>(
  items: T[],
  fn: (item: T) => Promise<R>,
  concurrency: number
): Promise<R[]> {
  const results: R[] = [];
  const executing = new Set<Promise<void>>();

  for (const [index, item] of items.entries()) {
    const p = fn(item).then(result => {
      results[index] = result;
    });
    executing.add(p);
    p.finally(() => executing.delete(p));

    if (executing.size >= concurrency) {
      await Promise.race(executing);
    }
  }

  await Promise.all(executing);
  return results;
}

// Use: process 100 items, max 5 at a time
const results = await pMap(urls, url => fetch(url), 5);
```

## Streams

Streams represent data that arrives in chunks over time. This is critical for AI work — LLM responses are streamed token by token.

```typescript
// Reading a stream with async iteration
const response = await fetch("https://api.example.com/large-data");
const reader = response.body!.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  // `{ stream: true }` tells the decoder more data is coming.
  // Without it, multi-byte UTF-8 characters split across chunk boundaries
  // would be decoded incorrectly. Always pass this option inside a loop.
  const text = decoder.decode(value, { stream: true });
  process.stdout.write(text);  // Print without newline
}
```

### Async Iterables

```typescript
// for await...of — like Python's async for
async function* generateNumbers(): AsyncGenerator<number> {
  for (let i = 0; i < 5; i++) {
    await new Promise(resolve => setTimeout(resolve, 100));
    yield i;
  }
}

for await (const num of generateNumbers()) {
  console.log(num);  // 0, 1, 2, 3, 4 — one every 100ms
}
```

**Python comparison:**

```python
async def generate_numbers():
    for i in range(5):
        await asyncio.sleep(0.1)
        yield i

async for num in generate_numbers():
    print(num)
```

Nearly identical syntax. You'll use `for await` heavily when streaming AI model responses.

### Server-Sent Events (SSE)

AI APIs (Anthropic, OpenAI, etc.) stream responses using **Server-Sent Events** — a simple text format where each event is a `data:` line followed by a blank line:

```
data: {"type":"content","token":"Hello"}

data: {"type":"content","token":" world"}

data: [DONE]
```

The SDK handles SSE parsing for you, but understanding the format helps when debugging or building custom integrations. Parsing it is straightforward:

```typescript
function parseSSELine(line: string): unknown | null {
  if (!line.startsWith("data: ")) return null;  // Skip non-data lines
  const payload = line.slice(6);                 // Remove "data: " prefix
  if (payload === "[DONE]") return null;         // Sentinel value — stream is finished
  return JSON.parse(payload);
}
```

**Python comparison:** Python's `httpx` exposes SSE similarly — you iterate lines and check for `data:` prefixes. The format is identical; only the parsing code differs.

## Timers

```typescript
// Run after delay (like asyncio.call_later)
setTimeout(() => console.log("delayed"), 1000);

// Run repeatedly (no direct Python equivalent)
const intervalId = setInterval(() => console.log("tick"), 1000);
clearInterval(intervalId);  // Stop it

// Awaitable sleep (Python's asyncio.sleep equivalent)
// `resolve` is the fulfillment callback the Promise passes to your executor.
// Giving it directly to `setTimeout` means: "call resolve() after ms milliseconds."
// When resolve() is called with no argument, the promise fulfills with `undefined`.
const sleep = (ms: number) => new Promise<void>(resolve => setTimeout(resolve, ms));
await sleep(1000);  // Wait 1 second
```

There is no synchronous `time.sleep()` in JavaScript. Blocking the event loop freezes everything.

## AbortController — Cancellation

```typescript
// Cancel an in-flight request (like Python's asyncio.Task.cancel())
const controller = new AbortController();

// Start a fetch with a signal
const fetchPromise = fetch(url, { signal: controller.signal });

// Cancel it after 5 seconds
setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetchPromise;
} catch (error) {
  if (error instanceof DOMException && error.name === "AbortError") {
    console.log("Request was cancelled");
  }
}
```

## Further Reading

- [javascript.info — Promises, async/await](https://javascript.info/async) — Thorough async tutorial
- [MDN — Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) — Reference
- [Node.js Streams](https://nodejs.org/api/stream.html) — Node stream documentation

## Exercises

1. [Promises Basics](./exercises/01-promises-basics.md) — Convert callbacks to Promises and async/await
2. [Async Data Fetcher](./exercises/02-async-await.md) — Fetch data from a real API
3. [Parallel Requests](./exercises/03-parallel-requests.md) — Build a concurrent fetcher with limits
4. [Streaming](./exercises/04-streaming.md) — Process streaming responses

Next: [Module 04 — Modules and Packages](../04-modules-and-packages/README.md)
