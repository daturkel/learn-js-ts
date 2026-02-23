# Module 08: Web Servers and APIs

Build HTTP servers and REST APIs with Hono, compared to Python's FastAPI.

**Time:** 2-3 sessions

## Hono — Like FastAPI for JavaScript

Hono is a modern, TypeScript-first web framework. It's lightweight, fast, and has excellent type inference. Think of it as the FastAPI of the JS world.

```bash
npm install hono @hono/node-server
```

### Hello World — Side by Side

```python
# Python (FastAPI)
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello, World!"}

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"id": user_id, "name": f"User {user_id}"}
```

```typescript
// TypeScript (Hono)
import { Hono } from "hono";
import { serve } from "@hono/node-server";

const app = new Hono();

app.get("/", (c) => {
  return c.json({ message: "Hello, World!" });
});

app.get("/users/:id", (c) => {
  const id = c.req.param("id");
  return c.json({ id, name: `User ${id}` });
});

serve({ fetch: app.fetch, port: 3000 });
console.log("Server running on http://localhost:3000");
```

Key differences:
- Hono uses `app.get()` / `app.post()` instead of `@app.get()` decorators
- Route params use `:id` syntax instead of `{id}`
- The handler receives a context `c` (not individual params)
- `c.json()` returns a JSON response

## Routes and Parameters

```typescript
const app = new Hono();

// Path parameters (like FastAPI's {param})
app.get("/experiments/:id", (c) => {
  const id = c.req.param("id");
  return c.json({ id });
});

// Query parameters (like FastAPI's Query)
app.get("/search", (c) => {
  const query = c.req.query("q") ?? "";
  const limit = parseInt(c.req.query("limit") ?? "10");
  return c.json({ query, limit });
});
// GET /search?q=bert&limit=5

// Multiple methods
app.post("/experiments", async (c) => {
  const body = await c.req.json();
  return c.json({ created: body }, 201);
});

app.put("/experiments/:id", async (c) => {
  const id = c.req.param("id");
  const body = await c.req.json();
  return c.json({ updated: { id, ...body } });
});

app.delete("/experiments/:id", (c) => {
  const id = c.req.param("id");
  return c.json({ deleted: id });
});
```

## Request and Response

```typescript
// Reading the request
app.post("/data", async (c) => {
  const body = await c.req.json();           // Parse JSON body
  const contentType = c.req.header("content-type");  // Read header
  const auth = c.req.header("authorization");

  // Returning responses
  return c.json({ data: body }, 201);        // JSON with status code
  // return c.text("OK");                    // Plain text
  // return c.html("<h1>Hello</h1>");        // HTML
  // return c.redirect("/other");            // Redirect
  // return c.body(null, 204);               // No content
});
```

## Middleware

Middleware functions run before/after route handlers — like FastAPI middleware or Python decorators:

```typescript
import { Hono } from "hono";
import { logger } from "hono/logger";
import { cors } from "hono/cors";

const app = new Hono();

// Built-in middleware
app.use("*", logger());                      // Log all requests
app.use("*", cors());                        // Enable CORS

// Custom middleware
app.use("*", async (c, next) => {
  const start = Date.now();
  await next();                              // Call the next handler
  const duration = Date.now() - start;
  c.header("X-Response-Time", `${duration}ms`);
  console.log(`${c.req.method} ${c.req.path} — ${duration}ms`);
});

// Auth middleware (apply to specific routes)
const requireAuth = async (c, next) => {
  const apiKey = c.req.header("x-api-key");
  if (apiKey !== "secret") {
    return c.json({ error: "Unauthorized" }, 401);
  }
  await next();
};

app.get("/public", (c) => c.json({ data: "anyone can see" }));
app.get("/private", requireAuth, (c) => c.json({ data: "secret stuff" }));
```

## Input Validation with Zod

Zod is the Pydantic of the JS world — runtime type validation:

```bash
npm install zod
```

```python
# Python (Pydantic + FastAPI)
from pydantic import BaseModel

class CreateExperiment(BaseModel):
    name: str
    model: str
    learning_rate: float = 0.001

@app.post("/experiments")
def create(data: CreateExperiment):
    return data.dict()
```

```typescript
// TypeScript (Zod + Hono)
import { z } from "zod";

const CreateExperimentSchema = z.object({
  name: z.string().min(1),
  model: z.string(),
  learningRate: z.number().positive().default(0.001),
});

type CreateExperiment = z.infer<typeof CreateExperimentSchema>;

app.post("/experiments", async (c) => {
  const body = await c.req.json();
  const result = CreateExperimentSchema.safeParse(body);

  if (!result.success) {
    return c.json({ error: result.error.issues }, 400);
  }

  const experiment: CreateExperiment = result.data;
  return c.json(experiment, 201);
});
```

`z.infer<typeof Schema>` extracts the TypeScript type from the Zod schema — so you define the schema once and get both runtime validation AND compile-time types. Pydantic does this too, but it's built into the class definition.

## Error Handling

```typescript
import { HTTPException } from "hono/http-exception";

// Throw HTTP errors
app.get("/experiments/:id", (c) => {
  const id = c.req.param("id");
  const experiment = db.get(id);
  if (!experiment) {
    throw new HTTPException(404, { message: `Experiment ${id} not found` });
  }
  return c.json(experiment);
});

// Global error handler
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return c.json({ error: err.message }, err.status);
  }
  console.error("Unexpected error:", err);
  return c.json({ error: "Internal server error" }, 500);
});
```

## Brief: Express.js

Express is the older standard (like Flask to FastAPI). You'll see it in many existing projects:

```typescript
// Express (for reference — don't use for new projects)
import express from "express";
const app = express();

app.use(express.json());

app.get("/", (req, res) => {
  res.json({ message: "Hello" });
});

app.listen(3000);
```

Hono advantages over Express: TypeScript-first, faster, modern API, works on edge runtimes.

## Further Reading

- [Hono Documentation](https://hono.dev/) — Full reference
- [Zod Documentation](https://zod.dev/) — Schema validation
- [Express.js Guide](https://expressjs.com/en/guide/routing.html) — For reading existing Express code

## Exercises

1. [Hello Server](./exercises/01-hello-server.md) — Basic Hono routes
2. [REST API](./exercises/02-rest-api.md) — CRUD API for experiments
3. [Middleware](./exercises/03-middleware.md) — Custom middleware

---

**Next:** [Exercise 1: Hello Server →](/08-web-servers-and-apis/exercises/01-hello-server.md)
