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

### Task 2: Fetch multiple users

Fetch users 1 through 5 concurrently using `Promise.all`. Print each user's name.

### Task 3: Error handling

Write a robust `safeFetch<T>` wrapper that:
- Returns `{ ok: true, data: T }` on success
- Returns `{ ok: false, error: string }` on failure
- Handles: network errors, non-200 status codes, JSON parse errors
- Takes a URL and returns a typed result

Test it with a valid URL and an invalid one (e.g., `https://jsonplaceholder.typicode.com/users/99999`).

### Task 4: Compare to Python

Here's the Python equivalent using httpx. Note the similarities:

```python
import httpx
import asyncio

async def fetch_user(user_id: int) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://jsonplaceholder.typicode.com/users/{user_id}")
        response.raise_for_status()
        return response.json()

async def main():
    users = await asyncio.gather(*[fetch_user(i) for i in range(1, 6)])
    for user in users:
        print(user["name"])

asyncio.run(main())
```

Notice: Python needs `asyncio.run()` to start the event loop. JavaScript doesn't — async code runs natively.

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
```

### Key takeaways
- `fetch` is built in (no npm package needed, unlike Python which needs `httpx` or `requests`)
- `fetch` does NOT throw on 4xx/5xx — you must check `response.ok`
- `response.json()` returns a Promise (must be awaited)
- `as Promise<User>` is a type assertion — `fetch` returns `any`, so you tell TS what to expect
- `Promise.all` with `.map()` is the standard pattern for parallel async operations
</details>
