# Exercise: Parallel Requests with Concurrency Limit

Fetch many URLs concurrently, but limit how many run at once.

## Goal

Build a concurrency-limited parallel fetcher — like a semaphore. This is a common real-world pattern: you have 100 URLs to fetch, but you don't want to slam the server with 100 simultaneous requests.

## Background

Firing off a hundred requests at once with `Promise.all` is fast, but it can overwhelm a server, hit rate limits, or exhaust local memory with pending responses. The solution is a **concurrency pool**: keep at most N operations running at any moment, and as soon as one finishes, start the next.

JavaScript has no built-in primitive for this. You'll build one from `Promise.race` — which resolves as soon as *any* Promise in a set settles, effectively freeing a slot in the pool.

## Setup

Create `parallel.ts`. We'll use JSONPlaceholder endpoints as our test URLs.

**Two utilities you'll use throughout this exercise:**

`performance.now()` — returns a high-resolution timestamp in milliseconds. Take two readings and subtract to measure elapsed time. It's a browser/Node.js global; no import needed.

`Array.from({ length: n }, (_, i) => expr)` — JS's equivalent of Python's `range()`. Creates an array of `n` elements using a mapping function. The first argument `_` is the unused element value; `i` is the 0-based index.

```typescript
Array.from({ length: 10 }, (_, i) => `posts/${i + 1}`)
// => ["posts/1", "posts/2", ..., "posts/10"]
```

## Tasks

### Task 1: Sequential baseline

Fetch 10 posts from JSONPlaceholder sequentially. Measure total time.

```typescript
const urls = Array.from({ length: 10 }, (_, i) =>
  `https://jsonplaceholder.typicode.com/posts/${i + 1}`
);
```

Time it with `performance.now()`:

```typescript
const start = performance.now();
// ... fetch all sequentially
const elapsed = performance.now() - start;
console.log(`Sequential: ${elapsed.toFixed(0)}ms`);
```

### Task 2: Unlimited parallel

Fetch all 10 with `Promise.all` (no limit). Measure time. Should be much faster.

### Task 3: Limited concurrency

Implement `pMap` — a function that runs async operations with a concurrency limit:

```typescript
async function pMap<T, R>(
  items: T[],
  fn: (item: T) => Promise<R>,
  concurrency: number
): Promise<R[]> {
  // Your implementation here
}
```

Test with concurrency of 3 on the same 10 URLs. Time should be between sequential and unlimited.

### Task 4: Add progress reporting

Modify `pMap` to accept an optional `onProgress` callback that fires after each item completes:

```typescript
await pMap(
  urls,
  url => fetchPost(url),
  3,
  (completed, total) => {
    console.log(`Progress: ${completed}/${total}`);
  }
);
```

<details>
<summary>Hint: pMap implementation approach</summary>

The trick is to maintain a "pool" of running promises. When one finishes, start the next:

1. Keep a `Set` of currently executing promises
2. For each item, start the async operation and add its promise to the set
3. When the set reaches `concurrency` size, `await Promise.race(executing)` to wait for one to finish
4. After the loop, `await Promise.all(executing)` to drain remaining
</details>

<details>
<summary>Solution</summary>

```typescript
interface Post {
  id: number;
  title: string;
  body: string;
  userId: number;
}

async function fetchPost(url: string): Promise<Post> {
  const response = await fetch(url);
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json() as Promise<Post>;
}

const urls = Array.from({ length: 10 }, (_, i) =>
  `https://jsonplaceholder.typicode.com/posts/${i + 1}`
);

// --- Task 1: Sequential ---

async function sequential() {
  const start = performance.now();
  const results: Post[] = [];

  for (const url of urls) {
    results.push(await fetchPost(url));
  }

  const elapsed = performance.now() - start;
  console.log(`Sequential: ${elapsed.toFixed(0)}ms (${results.length} posts)`);
  return results;
}

// --- Task 2: Unlimited parallel ---

async function unlimited() {
  const start = performance.now();
  const results = await Promise.all(urls.map(url => fetchPost(url)));
  const elapsed = performance.now() - start;
  console.log(`Unlimited parallel: ${elapsed.toFixed(0)}ms (${results.length} posts)`);
  return results;
}

// --- Task 3: Limited concurrency ---

async function pMap<T, R>(
  items: T[],
  fn: (item: T) => Promise<R>,
  concurrency: number,
  onProgress?: (completed: number, total: number) => void
): Promise<R[]> {
  const results: R[] = new Array(items.length);
  let completed = 0;
  const executing = new Set<Promise<void>>();

  for (const [index, item] of items.entries()) {
    const p = fn(item).then(result => {
      results[index] = result;
      completed++;
      onProgress?.(completed, items.length);
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

async function limited() {
  const start = performance.now();
  const results = await pMap(urls, url => fetchPost(url), 3);
  const elapsed = performance.now() - start;
  console.log(`Limited (3): ${elapsed.toFixed(0)}ms (${results.length} posts)`);
  return results;
}

// --- Task 4: With progress ---

async function withProgress() {
  console.log("\nWith progress:");
  const results = await pMap(
    urls,
    url => fetchPost(url),
    3,
    (done, total) => console.log(`  ${done}/${total} complete`)
  );
  console.log(`Fetched ${results.length} posts`);
}

// --- Run all ---

await sequential();
await unlimited();
await limited();
await withProgress();
```

### Expected output (times will vary)

```
Sequential: ~2000ms (10 posts)
Unlimited parallel: ~200ms (10 posts)
Limited (3): ~800ms (10 posts)

With progress:
  1/10 complete
  2/10 complete
  ...
  10/10 complete
Fetched 10 posts
```

### How pMap works

The key is `Promise.race(executing)`. It resolves as soon as ANY promise in the set finishes. This frees up a "slot" for the next item. The result:

- At most `concurrency` operations run simultaneously
- As soon as one finishes, the next starts immediately
- Order of results is preserved (using the `index`)

This pattern — a fixed-size pool that refills as work completes — is a fundamental async concurrency primitive, and here you've built it from scratch using only `Promise.race`.
</details>

---

**Next:** [Exercise 4: Streaming →](/03-async-patterns/exercises/04-streaming.md)
