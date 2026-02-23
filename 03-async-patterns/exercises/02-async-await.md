# Exercise: Async Data Fetcher

Fetch data from a real public API using `async/await`.

## Goal

Build a typed async function that fetches data from [JSONPlaceholder](https://jsonplaceholder.typicode.com/) (a free fake REST API). Practice error handling, JSON parsing, and type safety with `fetch`.

## Setup

Create `fetcher.ts`. No npm packages needed — `fetch` is built into Node 18+.

## Tasks

### Task 1: Fetch a single user

Fetch `https://jsonplaceholder.typicode.com/users/1` and print the user's name and email.

- Define a `User` interface based on the API response
- Handle network errors and non-200 responses
- Parse and return the typed result

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  username: string;
  // ... (you can add more fields or keep it minimal)
}

async function fetchUser(id: number): Promise<User> {
  // Your code here
}
```

**Note on type safety:** When you cast `response.json()` to `Promise<User>`, TypeScript takes your word for it. There's no runtime check that the API actually returned the right shape — if it returns something unexpected, TypeScript won't warn you and your code will silently break when it tries to access a missing field. This is a fundamental limitation: TypeScript types are erased at runtime, so external data (API responses, JSON files, user input) is always an unverified assertion at the boundary. Runtime validation with Zod (covered in Module 08) is how you make that boundary safe in production code.

### Task 2: Fetch multiple users

Fetch users 1 through 5 concurrently using `Promise.all`. Print each user's name.

### Task 3: Error handling

Write a robust `safeFetch<T>` wrapper that:
- Returns `{ ok: true, data: T }` on success
- Returns `{ ok: false, error: string }` on failure
- Handles: network errors, non-200 status codes, JSON parse errors
- Takes a URL and returns a typed result

Test it with a valid URL and an invalid one (e.g., `https://jsonplaceholder.typicode.com/users/99999`).

### Task 4: Add retry logic

Network requests fail. Write a `fetchWithRetry` wrapper that retries a failed request up to `maxRetries` times before giving up:

```typescript
async function fetchWithRetry<T>(
  url: string,
  maxRetries: number
): Promise<T> {
  // Your code here
}
```

Requirements:
- Retry only on network errors or 5xx responses (not 4xx — those are client errors that won't get better on retry)
- Throw after all retries are exhausted
- Test it by temporarily using a bad URL to trigger failures

This is the pattern you'll reach for when calling external APIs that occasionally return transient errors.

<details>
<summary>Solution</summary>

```typescript
// --- Types ---

interface User {
  id: number;
  name: string;
  email: string;
  username: string;
  phone: string;
  website: string;
}

type Result<T> =
  | { ok: true; data: T }
  | { ok: false; error: string };

// --- Task 1: Fetch single user ---

async function fetchUser(id: number): Promise<User> {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/users/${id}`
  );

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }

  return response.json() as Promise<User>;
}

// Test Task 1
const user = await fetchUser(1);
console.log(`User: ${user.name} (${user.email})`);

// --- Task 2: Fetch multiple users ---

const userIds = [1, 2, 3, 4, 5];
const users = await Promise.all(userIds.map(id => fetchUser(id)));

console.log("\nAll users:");
for (const u of users) {
  console.log(`  ${u.id}: ${u.name}`);
}

// --- Task 3: Safe fetch wrapper ---

async function safeFetch<T>(url: string): Promise<Result<T>> {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      return { ok: false, error: `HTTP ${response.status}: ${response.statusText}` };
    }

    const data = await response.json() as T;
    return { ok: true, data };
  } catch (error) {
    const message = error instanceof Error ? error.message : String(error);
    return { ok: false, error: message };
  }
}

// Test with valid URL
const result1 = await safeFetch<User>(
  "https://jsonplaceholder.typicode.com/users/1"
);
if (result1.ok) {
  console.log(`\nSafe fetch success: ${result1.data.name}`);
} else {
  console.log(`\nSafe fetch failed: ${result1.error}`);
}

// Test with invalid URL
const result2 = await safeFetch<User>(
  "https://jsonplaceholder.typicode.com/users/99999"
);
if (result2.ok) {
  console.log(`Safe fetch success: ${result2.data.name}`);
} else {
  console.log(`Safe fetch failed: ${result2.error}`);
  // "HTTP 404: Not Found"
}

// --- Task 4: Retry wrapper ---

async function fetchWithRetry<T>(url: string, maxRetries: number): Promise<T> {
  let lastError: Error | undefined;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url);

      // 4xx errors are client mistakes — retrying won't help
      if (response.status >= 400 && response.status < 500) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      // 5xx errors are server-side — worth retrying
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      return await response.json() as T;
    } catch (error) {
      lastError = error instanceof Error ? error : new Error(String(error));

      // Don't retry 4xx errors
      if (lastError.message.match(/^HTTP 4\d\d/)) throw lastError;

      if (attempt < maxRetries) {
        console.log(`Attempt ${attempt + 1} failed, retrying...`);
      }
    }
  }

  throw new Error(`Failed after ${maxRetries + 1} attempts: ${lastError?.message}`);
}

// Test
const user = await fetchWithRetry<User>(
  "https://jsonplaceholder.typicode.com/users/1",
  3
);
console.log(`\nFetch with retry: ${user.name}`);
```

### Key takeaways
- `fetch` does NOT throw on 4xx/5xx — you must check `response.ok` or `response.status`
- `response.json()` returns a Promise — it must be awaited separately from `fetch`
- `as Promise<User>` is a type assertion: `fetch` returns `any`, so you tell TS what to expect
- `Promise.all` with `.map()` is the standard pattern for launching parallel async operations
- Retry logic should distinguish transient server errors (5xx) from permanent client errors (4xx)
</details>

---

**Next:** [Exercise 3: Parallel Requests →](/03-async-patterns/exercises/03-parallel-requests.md)
