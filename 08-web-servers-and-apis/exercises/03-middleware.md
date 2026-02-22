# Exercise: Custom Middleware

Write four custom middleware functions for a Hono server.

## Setup

Use the same project from the previous exercise, or start fresh.

## Tasks

Create these middleware functions:

### 1. Request ID Middleware

Add a unique ID to every request/response:
- Generate a UUID for each request
- Add it as `X-Request-Id` response header
- Make it available to route handlers

### 2. Timing Middleware

Log how long each request takes:
- Record start time before handling
- Log method, path, status code, and duration after handling

### 3. Auth Middleware

Check for an API key in the `x-api-key` header:
- If missing or wrong, return 401
- If correct, proceed to the handler
- Apply only to `/api/*` routes (not `/health`)

### 4. Error Handler

Catch all errors and return consistent JSON:
- `{ "error": "message", "requestId": "..." }`
- 500 for unexpected errors
- Preserve status code for HTTP exceptions

<details>
<summary>Solution</summary>

```typescript
import { Hono } from "hono";
import { serve } from "@hono/node-server";
import { randomUUID } from "node:crypto";

const app = new Hono();

// --- 1. Request ID ---

app.use("*", async (c, next) => {
  const requestId = randomUUID();
  c.set("requestId", requestId);
  await next();
  c.header("X-Request-Id", requestId);
});

// --- 2. Timing ---

app.use("*", async (c, next) => {
  const start = Date.now();
  await next();
  const duration = Date.now() - start;
  const requestId = c.get("requestId") ?? "unknown";
  console.log(
    `[${requestId.slice(0, 8)}] ${c.req.method} ${c.req.path} → ${c.res.status} (${duration}ms)`
  );
});

// --- 3. Auth (only for /api/*) ---

const API_KEY = "sk-test-12345";

app.use("/api/*", async (c, next) => {
  const apiKey = c.req.header("x-api-key");
  if (apiKey !== API_KEY) {
    return c.json(
      { error: "Unauthorized", requestId: c.get("requestId") },
      401
    );
  }
  await next();
});

// --- 4. Error handler ---

app.onError((err, c) => {
  const requestId = c.get("requestId") ?? "unknown";
  console.error(`[${requestId.slice(0, 8)}] Error:`, err.message);

  const status = "status" in err ? (err as any).status : 500;
  return c.json(
    {
      error: status === 500 ? "Internal server error" : err.message,
      requestId,
    },
    status
  );
});

// --- Routes ---

app.get("/health", (c) => {
  return c.json({ status: "ok" });
});

app.get("/api/data", (c) => {
  return c.json({ data: "secret stuff", requestId: c.get("requestId") });
});

app.get("/api/error", (c) => {
  throw new Error("Something broke!");
});

serve({ fetch: app.fetch, port: 3000 });
console.log("Server running on http://localhost:3000");
```

Test:
```bash
# Health (no auth needed)
curl -i http://localhost:3000/health

# API without auth (should 401)
curl -i http://localhost:3000/api/data

# API with auth
curl -i -H "x-api-key: sk-test-12345" http://localhost:3000/api/data

# Error endpoint
curl -i -H "x-api-key: sk-test-12345" http://localhost:3000/api/error
```

Check the response headers — you'll see `X-Request-Id` on every response, and the server logs will show timing.
</details>
