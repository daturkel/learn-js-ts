# Exercise: REST API for ML Experiments

Build a CRUD REST API with Zod validation.

## Setup

```bash
mkdir experiment-api && cd experiment-api
npm init -y
npm install hono @hono/node-server zod
npm install -D typescript tsx
```

## Tasks

Build an API that manages ML experiments in memory (no database).

### Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST /api/experiments` | Create an experiment |
| `GET /api/experiments` | List all experiments |
| `GET /api/experiments/:id` | Get one experiment |
| `PUT /api/experiments/:id` | Update an experiment |
| `DELETE /api/experiments/:id` | Delete an experiment |

### Data Model

```typescript
interface Experiment {
  id: string;
  name: string;
  model: string;
  hyperparameters: Record<string, number>;
  status: "draft" | "running" | "completed" | "failed";
  createdAt: string;
  updatedAt: string;
}
```

### Validation with Zod

Create request body:
- `name` — required, non-empty string
- `model` — required, non-empty string
- `hyperparameters` — optional, defaults to `{}`
- `status` — optional, defaults to `"draft"`, must be one of the valid statuses

Update request body: all fields optional (partial update).

### Error Responses

- `404` for missing experiments
- `400` for validation errors (return Zod error details)

<details>
<summary>Hint: Generating IDs</summary>

```typescript
import { randomUUID } from "node:crypto";
const id = randomUUID();
```
</details>

<details>
<summary>Hint: In-memory storage</summary>

```typescript
const experiments = new Map<string, Experiment>();
```
</details>

<details>
<summary>Solution</summary>

```typescript
import { Hono } from "hono";
import { logger } from "hono/logger";
import { serve } from "@hono/node-server";
import { z } from "zod";
import { randomUUID } from "node:crypto";

// --- Types ---

const StatusEnum = z.enum(["draft", "running", "completed", "failed"]);

const CreateExperimentSchema = z.object({
  name: z.string().min(1, "Name is required"),
  model: z.string().min(1, "Model is required"),
  hyperparameters: z.record(z.number()).default({}),
  status: StatusEnum.default("draft"),
});

const UpdateExperimentSchema = z.object({
  name: z.string().min(1).optional(),
  model: z.string().min(1).optional(),
  hyperparameters: z.record(z.number()).optional(),
  status: StatusEnum.optional(),
});

interface Experiment {
  id: string;
  name: string;
  model: string;
  hyperparameters: Record<string, number>;
  status: z.infer<typeof StatusEnum>;
  createdAt: string;
  updatedAt: string;
}

// --- Storage ---

const experiments = new Map<string, Experiment>();

// Seed with sample data
const seed: Omit<Experiment, "id" | "createdAt" | "updatedAt">[] = [
  { name: "BERT Sentiment", model: "bert-base", hyperparameters: { lr: 3e-5, epochs: 3 }, status: "completed" },
  { name: "GPT-2 Generation", model: "gpt2", hyperparameters: { lr: 5e-5, epochs: 5 }, status: "running" },
];
for (const s of seed) {
  const id = randomUUID();
  const now = new Date().toISOString();
  experiments.set(id, { id, ...s, createdAt: now, updatedAt: now });
}

// --- App ---

const app = new Hono();
app.use("*", logger());

// List all
app.get("/api/experiments", (c) => {
  const list = Array.from(experiments.values());
  return c.json(list);
});

// Get one
app.get("/api/experiments/:id", (c) => {
  const id = c.req.param("id");
  const exp = experiments.get(id);
  if (!exp) {
    return c.json({ error: `Experiment ${id} not found` }, 404);
  }
  return c.json(exp);
});

// Create
app.post("/api/experiments", async (c) => {
  const body = await c.req.json();
  const result = CreateExperimentSchema.safeParse(body);

  if (!result.success) {
    return c.json({ error: "Validation failed", details: result.error.issues }, 400);
  }

  const id = randomUUID();
  const now = new Date().toISOString();
  const experiment: Experiment = {
    id,
    ...result.data,
    createdAt: now,
    updatedAt: now,
  };

  experiments.set(id, experiment);
  return c.json(experiment, 201);
});

// Update
app.put("/api/experiments/:id", async (c) => {
  const id = c.req.param("id");
  const existing = experiments.get(id);
  if (!existing) {
    return c.json({ error: `Experiment ${id} not found` }, 404);
  }

  const body = await c.req.json();
  const result = UpdateExperimentSchema.safeParse(body);

  if (!result.success) {
    return c.json({ error: "Validation failed", details: result.error.issues }, 400);
  }

  const updated: Experiment = {
    ...existing,
    ...result.data,
    updatedAt: new Date().toISOString(),
  };

  experiments.set(id, updated);
  return c.json(updated);
});

// Delete
app.delete("/api/experiments/:id", (c) => {
  const id = c.req.param("id");
  if (!experiments.has(id)) {
    return c.json({ error: `Experiment ${id} not found` }, 404);
  }
  experiments.delete(id);
  return c.json({ deleted: id });
});

serve({ fetch: app.fetch, port: 3000 });
console.log("API running on http://localhost:3000");
```

Test with curl:
```bash
# List
curl http://localhost:3000/api/experiments | jq

# Create
curl -X POST http://localhost:3000/api/experiments \
  -H "Content-Type: application/json" \
  -d '{"name":"T5 Test","model":"t5-base","hyperparameters":{"lr":0.001}}' | jq

# Update (use an ID from the list response)
curl -X PUT http://localhost:3000/api/experiments/<id> \
  -H "Content-Type: application/json" \
  -d '{"status":"completed"}' | jq

# Invalid input
curl -X POST http://localhost:3000/api/experiments \
  -H "Content-Type: application/json" \
  -d '{"name":""}' | jq
```
</details>

---

**Next:** [Exercise 3: Middleware →](/08-web-servers-and-apis/exercises/03-middleware.md)
