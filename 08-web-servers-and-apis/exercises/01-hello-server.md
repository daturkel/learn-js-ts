# Exercise: Hello Server

Create a basic Hono web server with a few routes.

## Setup

```bash
mkdir hello-server && cd hello-server
npm init -y
npm install hono @hono/node-server
npm install -D typescript tsx
```

Add to `package.json`: `"type": "module"`

## Tasks

Create `src/index.ts` with these routes:

1. `GET /` — returns `{ "message": "Hello from Hono!" }`
2. `GET /health` — returns `{ "status": "ok", "uptime": <seconds since start> }`
3. `GET /echo?message=hello` — returns `{ "echo": "hello" }`. If no message query param, return `{ "echo": "(empty)" }`
4. `GET /greet/:name` — returns `{ "greeting": "Hello, <name>!" }`
5. Add the built-in logger middleware

Run with `npx tsx src/index.ts` and test with `curl`:

```bash
curl http://localhost:3000/
curl http://localhost:3000/health
curl http://localhost:3000/echo?message=testing
curl http://localhost:3000/greet/Alice
```

<details>
<summary>Solution</summary>

```typescript
import { Hono } from "hono";
import { logger } from "hono/logger";
import { serve } from "@hono/node-server";

const app = new Hono();
const startTime = Date.now();

app.use("*", logger());

app.get("/", (c) => {
  return c.json({ message: "Hello from Hono!" });
});

app.get("/health", (c) => {
  const uptimeSeconds = Math.floor((Date.now() - startTime) / 1000);
  return c.json({ status: "ok", uptime: uptimeSeconds });
});

app.get("/echo", (c) => {
  const message = c.req.query("message") ?? "(empty)";
  return c.json({ echo: message });
});

app.get("/greet/:name", (c) => {
  const name = c.req.param("name");
  return c.json({ greeting: `Hello, ${name}!` });
});

serve({ fetch: app.fetch, port: 3000 });
console.log("Server running on http://localhost:3000");
```

### FastAPI equivalent

```python
from fastapi import FastAPI, Query
import time

app = FastAPI()
start_time = time.time()

@app.get("/")
def root():
    return {"message": "Hello from FastAPI!"}

@app.get("/health")
def health():
    uptime = int(time.time() - start_time)
    return {"status": "ok", "uptime": uptime}

@app.get("/echo")
def echo(message: str = Query("(empty)")):
    return {"echo": message}

@app.get("/greet/{name}")
def greet(name: str):
    return {"greeting": f"Hello, {name}!"}
```

The structure is very similar. Main syntactic differences:
- Hono: `app.get("/path", handler)` vs FastAPI: `@app.get("/path")`
- Hono: `c.req.query("key")` vs FastAPI: function params with `Query()`
- Hono: `c.req.param("key")` vs FastAPI: function params matching `{path_param}`
</details>

---

**Next:** [Exercise 2: REST API →](/08-web-servers-and-apis/exercises/02-rest-api.md)
