# Module 03: Async Patterns

The single most important conceptual module. JavaScript's async model is the biggest paradigm shift from Python.


## The Event Loop

JavaScript is single-threaded. There are no threads, no multiprocessing, no GIL. All concurrency happens through the **event loop** — a single thread that processes tasks from a queue.

Unlike languages where you can reach for threads or separate processes to run work simultaneously, the event loop is JavaScript's only execution model. Concurrency here means interleaving tasks on one thread — not running them in parallel. An `await` expression doesn't block the thread; it suspends the current async function and lets other queued tasks run until the awaited value is ready.

### How it works

```
┌─────────────────────────┐
│       Call Stack        │  ← Executes one function at a time
└──────────┬──────────────┘
           │ (when stack is empty)
           ▼
┌─────────────────────────┐
│     Microtask Queue     │  ← Promise callbacks (.then), queueMicrotask
└──────────┬──────────────┘
           │ (when microtasks are done)
           ▼
┌─────────────────────────┐
│       Task Queue        │  ← setTimeout callbacks, I/O callbacks
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

`async/await` is syntactic sugar over Promises that lets you write async code as if it were synchronous. Marking a function `async` makes it implicitly return a Promise. Inside that function, `await` suspends execution until the awaited Promise settles, then resumes with the resolved value — without blocking the event loop. Everything else queued up on the event loop continues running while you wait.

```typescript
async function fetchUser(userId: number): Promise<Record<string, unknown>> {
  const response = await fetch(`https://api.example.com/users/${userId}`);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}
```

If you've used Python's `async/await`, the syntax is nearly identical. A few differences worth knowing:

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

### Promise.all

Pass an array of Promises to `Promise.all` and it waits for all of them to resolve, returning an array of results in the same order. All the operations start concurrently — they're not run one at a time.

```typescript
const results = await Promise.all([
  fetchUser(1),
  fetchUser(2),
  fetchUser(3),
]);
// results is [user1, user2, user3]
```

**Fails fast:** If any Promise rejects, `Promise.all` rejects immediately and the other results are discarded.

### Promise.allSettled

Use `Promise.allSettled` when you want results from all operations regardless of whether some fail. It waits for every Promise to settle, then gives you an array where each entry is either `{ status: "fulfilled", value }` or `{ status: "rejected", reason }`.

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

Unlike `Promise.all`, `Promise.allSettled` never rejects. It always resolves once every input Promise has settled.

### Promise.race and Promise.any

Sometimes you want whichever result arrives first. `Promise.race` resolves (or rejects) as soon as any input Promise settles. `Promise.any` is similar but only resolves on the first *success* — it keeps waiting through rejections until either a success arrives or everything has failed.

```typescript
// First to complete (success or failure) wins
const fastest = await Promise.race([fetchFromServer1(), fetchFromServer2()]);

// First SUCCESS wins (ignores rejections until all fail)
const firstSuccess = await Promise.any([fetchFromServer1(), fetchFromServer2()]);
```

### Concurrency Limiting

JavaScript has no built-in concurrency limiter. When you're fetching a large batch of URLs or processing many items, unconstrained parallelism can overwhelm a server or run into rate limits. The pattern is to keep a pool of at most N running Promises, and start the next one as soon as a slot opens:

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

Streams represent data that arrives incrementally rather than all at once. Instead of waiting for an entire response body to download before doing anything with it, you process each chunk the moment it arrives. For large payloads this reduces time-to-first-byte dramatically. For AI model responses it's essential: generation can take several seconds, and streaming lets you display each token as soon as the model produces it rather than waiting for the entire reply.

The raw HTTP streaming API gives you a `ReadableStream` on `response.body`. You get a `reader` from it, call `reader.read()` in a loop to pull chunks, and use a `TextDecoder` to convert the binary chunks to strings:

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

An **async generator** is a function declared `async function*` that `yield`s values asynchronously. Calling it returns an `AsyncGenerator` object, which you consume with `for await...of`. At each iteration the loop pauses until the next `yield` resolves, then continues — giving you a clean way to process a sequence of async values without manually managing a reader loop.

```typescript
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

`for await...of` works with any object that implements the async iterable protocol — including Node.js streams and the SDK streaming responses you'll see in later modules. Python has identical `async for` / `async def ... yield` syntax with the same semantics.

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

The wire format is standardized — any HTTP client in any language sees the same `data:` lines. What differs is only how you parse them.

## Timers

Timer functions are globals in both the browser and Node.js — no import needed. `setTimeout` schedules a callback once after a delay; `setInterval` schedules it repeatedly. Neither is awaitable on its own, but wrapping `setTimeout` in a Promise gives you an async-friendly `sleep`:

```typescript
// Run after delay
setTimeout(() => console.log("delayed"), 1000);

// Run repeatedly
const intervalId = setInterval(() => console.log("tick"), 1000);
clearInterval(intervalId);  // Stop it

// Awaitable sleep:
// `resolve` is the fulfillment callback the Promise passes to your executor.
// Giving it directly to `setTimeout` means: "call resolve() after ms milliseconds."
// When resolve() is called with no argument, the promise fulfills with `undefined`.
const sleep = (ms: number) => new Promise<void>(resolve => setTimeout(resolve, ms));
await sleep(1000);  // Wait 1 second
```

There is no synchronous sleep in JavaScript. A blocking loop would freeze the event loop and prevent all other async work from running.

## AbortController — Cancellation

When you start an async operation and later need to cancel it — because a user navigated away, a timeout expired, or a newer request supersedes it — you use an `AbortController`. You create a controller, pass its `.signal` to the operation, and call `.abort()` to cancel. Any operation holding that signal will reject with an `AbortError`.

```typescript
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

---

**Next:** [Exercise 1: Promises Basics →](/03-async-patterns/exercises/01-promises-basics.md)
