# Exercise: Experiment Tracker MCP Server

Build an MCP server that manages ML experiments — a practical tool you could actually use with Claude Desktop or Cursor.

## Setup

```bash
mkdir mcp-experiments && cd mcp-experiments
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D typescript tsx
```

Add to `package.json`: `"type": "module"`

## Tasks

### 1. Data model

Store experiments in memory with this structure:

```typescript
interface Experiment {
  id: string;
  name: string;
  model: string;
  hyperparameters: Record<string, number>;
  metrics: Record<string, number>;
  status: "draft" | "running" | "completed" | "failed";
  notes: string;
  createdAt: string;
  updatedAt: string;
}
```

Seed with 3-4 sample experiments.

### 2. Tools

Implement these tools:

**`list_experiments`** — List all experiments with optional filtering
- Parameters: `status` (optional), `model` (optional)
- Returns a summary table

**`get_experiment`** — Get full details of one experiment
- Parameter: `id` (string)
- Returns all experiment data formatted as readable text

**`create_experiment`** — Create a new experiment
- Parameters: `name`, `model`, `hyperparameters` (optional), `notes` (optional)
- Returns the created experiment

**`update_experiment`** — Update an experiment's status or metrics
- Parameters: `id`, `status` (optional), `metrics` (optional), `notes` (optional)
- Returns the updated experiment

**`compare_experiments`** — Compare two experiments side by side
- Parameters: `id1`, `id2`
- Returns a formatted comparison of metrics and hyperparameters

**`best_experiment`** — Find the best experiment by a given metric
- Parameters: `metric` (string, e.g. "accuracy"), `top_n` (optional, default 3)
- Returns the top N experiments sorted by that metric

### 3. Resources

**`experiments://all`** — Full experiment list as JSON
**`experiments://summary`** — Summary statistics (total count, count by status, average accuracy)

### 4. Prompts

**`analyze-results`** — Generates a prompt asking the model to analyze all experiment results and suggest next steps
**`debug-experiment`** — Takes an experiment `id` and generates a prompt asking the model to help debug why it failed

<details>
<summary>Hint: Formatting tool output</summary>

MCP tool responses are text that the AI model reads. Make them human-readable:

```typescript
// Good — formatted for readability
const text = experiments
  .map(e => `• ${e.name} (${e.model}) — ${e.status}, accuracy: ${e.metrics.accuracy ?? "N/A"}`)
  .join("\n");

// Bad — raw JSON dump for simple lists
const text = JSON.stringify(experiments);
```

Use JSON for complex nested data, formatted text for summaries.

</details>

<details>
<summary>Solution</summary>

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import { randomUUID } from "node:crypto";

// --- Data model ---

interface Experiment {
  id: string;
  name: string;
  model: string;
  hyperparameters: Record<string, number>;
  metrics: Record<string, number>;
  status: "draft" | "running" | "completed" | "failed";
  notes: string;
  createdAt: string;
  updatedAt: string;
}

const StatusEnum = z.enum(["draft", "running", "completed", "failed"]);

const experiments = new Map<string, Experiment>();

// Seed data
const seedData: Omit<Experiment, "id" | "createdAt" | "updatedAt">[] = [
  {
    name: "BERT Sentiment Analysis",
    model: "bert-base-uncased",
    hyperparameters: { learning_rate: 3e-5, epochs: 3, batch_size: 32 },
    metrics: { accuracy: 0.92, f1: 0.90, loss: 0.21 },
    status: "completed",
    notes: "Good results on IMDB dataset",
  },
  {
    name: "GPT-2 Text Generation",
    model: "gpt2",
    hyperparameters: { learning_rate: 5e-5, epochs: 5, batch_size: 16 },
    metrics: { perplexity: 24.5, bleu: 0.35 },
    status: "completed",
    notes: "Generates coherent text but sometimes repetitive",
  },
  {
    name: "T5 Summarization",
    model: "t5-small",
    hyperparameters: { learning_rate: 1e-4, epochs: 10, batch_size: 8 },
    metrics: {},
    status: "running",
    notes: "Training in progress",
  },
  {
    name: "DistilBERT NER",
    model: "distilbert-base-uncased",
    hyperparameters: { learning_rate: 2e-5, epochs: 4, batch_size: 32 },
    metrics: { accuracy: 0.45, f1: 0.38, loss: 1.8 },
    status: "failed",
    notes: "Poor results — likely need more training data or different preprocessing",
  },
];

for (const data of seedData) {
  const id = randomUUID().slice(0, 8);
  const now = new Date().toISOString();
  experiments.set(id, { id, ...data, createdAt: now, updatedAt: now });
}

// --- Server setup ---

const server = new McpServer({
  name: "experiment-tracker",
  version: "1.0.0",
});

// --- Tools ---

server.tool(
  "list_experiments",
  "List all ML experiments, optionally filtered by status or model",
  {
    status: StatusEnum.optional().describe("Filter by status"),
    model: z.string().optional().describe("Filter by model name (partial match)"),
  },
  async ({ status, model }) => {
    let results = Array.from(experiments.values());

    if (status) results = results.filter((e) => e.status === status);
    if (model) results = results.filter((e) => e.model.toLowerCase().includes(model.toLowerCase()));

    if (results.length === 0) {
      return { content: [{ type: "text", text: "No experiments found matching the criteria." }] };
    }

    const lines = results.map((e) => {
      const metrics = Object.entries(e.metrics)
        .map(([k, v]) => `${k}: ${v}`)
        .join(", ");
      return `• [${e.id}] ${e.name} (${e.model}) — ${e.status}${metrics ? ` | ${metrics}` : ""}`;
    });

    return {
      content: [{ type: "text", text: `Found ${results.length} experiment(s):\n\n${lines.join("\n")}` }],
    };
  },
);

server.tool(
  "get_experiment",
  "Get full details of a specific experiment by ID",
  { id: z.string().describe("Experiment ID") },
  async ({ id }) => {
    const exp = experiments.get(id);
    if (!exp) {
      return { content: [{ type: "text", text: `Experiment "${id}" not found.` }], isError: true };
    }

    const text = [
      `# ${exp.name}`,
      `ID: ${exp.id}`,
      `Model: ${exp.model}`,
      `Status: ${exp.status}`,
      `Created: ${exp.createdAt}`,
      `Updated: ${exp.updatedAt}`,
      "",
      "## Hyperparameters",
      ...Object.entries(exp.hyperparameters).map(([k, v]) => `  ${k}: ${v}`),
      "",
      "## Metrics",
      ...(Object.keys(exp.metrics).length > 0
        ? Object.entries(exp.metrics).map(([k, v]) => `  ${k}: ${v}`)
        : ["  No metrics recorded yet"]),
      "",
      "## Notes",
      exp.notes || "No notes",
    ].join("\n");

    return { content: [{ type: "text", text }] };
  },
);

server.tool(
  "create_experiment",
  "Create a new ML experiment",
  {
    name: z.string().describe("Experiment name"),
    model: z.string().describe("Model name/identifier"),
    hyperparameters: z.record(z.number()).optional().describe("Hyperparameters as key-value pairs"),
    notes: z.string().optional().describe("Initial notes"),
  },
  async ({ name, model, hyperparameters, notes }) => {
    const id = randomUUID().slice(0, 8);
    const now = new Date().toISOString();

    const experiment: Experiment = {
      id,
      name,
      model,
      hyperparameters: hyperparameters ?? {},
      metrics: {},
      status: "draft",
      notes: notes ?? "",
      createdAt: now,
      updatedAt: now,
    };

    experiments.set(id, experiment);

    return {
      content: [{
        type: "text",
        text: `Created experiment "${name}" with ID: ${id}\nModel: ${model}\nStatus: draft`,
      }],
    };
  },
);

server.tool(
  "update_experiment",
  "Update an experiment's status, metrics, or notes",
  {
    id: z.string().describe("Experiment ID"),
    status: StatusEnum.optional().describe("New status"),
    metrics: z.record(z.number()).optional().describe("Metrics to add/update"),
    notes: z.string().optional().describe("Updated notes"),
  },
  async ({ id, status, metrics, notes }) => {
    const exp = experiments.get(id);
    if (!exp) {
      return { content: [{ type: "text", text: `Experiment "${id}" not found.` }], isError: true };
    }

    if (status) exp.status = status;
    if (metrics) Object.assign(exp.metrics, metrics);
    if (notes !== undefined) exp.notes = notes;
    exp.updatedAt = new Date().toISOString();

    const changes = [
      status && `status → ${status}`,
      metrics && `metrics updated: ${Object.keys(metrics).join(", ")}`,
      notes !== undefined && `notes updated`,
    ].filter(Boolean).join(", ");

    return {
      content: [{ type: "text", text: `Updated experiment "${exp.name}" (${id}): ${changes}` }],
    };
  },
);

server.tool(
  "compare_experiments",
  "Compare two experiments side by side",
  {
    id1: z.string().describe("First experiment ID"),
    id2: z.string().describe("Second experiment ID"),
  },
  async ({ id1, id2 }) => {
    const e1 = experiments.get(id1);
    const e2 = experiments.get(id2);

    if (!e1 || !e2) {
      const missing = [!e1 && id1, !e2 && id2].filter(Boolean).join(", ");
      return { content: [{ type: "text", text: `Experiment(s) not found: ${missing}` }], isError: true };
    }

    // Collect all metric keys
    const allMetrics = new Set([...Object.keys(e1.metrics), ...Object.keys(e2.metrics)]);

    const metricsComparison = Array.from(allMetrics).map((key) => {
      const v1 = e1.metrics[key];
      const v2 = e2.metrics[key];
      const winner = v1 !== undefined && v2 !== undefined
        ? (key === "loss" || key === "perplexity" ? (v1 < v2 ? "←" : "→") : (v1 > v2 ? "←" : "→"))
        : "";
      return `  ${key}: ${v1 ?? "N/A"} vs ${v2 ?? "N/A"} ${winner}`;
    });

    const text = [
      `# Comparison: ${e1.name} vs ${e2.name}`,
      "",
      `Models: ${e1.model} vs ${e2.model}`,
      `Status: ${e1.status} vs ${e2.status}`,
      "",
      "## Hyperparameters",
      ...Object.keys({ ...e1.hyperparameters, ...e2.hyperparameters }).map((key) =>
        `  ${key}: ${e1.hyperparameters[key] ?? "N/A"} vs ${e2.hyperparameters[key] ?? "N/A"}`
      ),
      "",
      "## Metrics (← = first is better, → = second is better)",
      ...(metricsComparison.length > 0 ? metricsComparison : ["  No metrics to compare"]),
    ].join("\n");

    return { content: [{ type: "text", text }] };
  },
);

server.tool(
  "best_experiment",
  "Find the best experiments by a specific metric",
  {
    metric: z.string().describe("Metric name (e.g. 'accuracy', 'f1', 'loss')"),
    top_n: z.number().optional().describe("Number of results to return (default: 3)"),
  },
  async ({ metric, top_n }) => {
    const n = top_n ?? 3;
    const isLowerBetter = ["loss", "perplexity"].includes(metric.toLowerCase());

    const withMetric = Array.from(experiments.values())
      .filter((e) => e.metrics[metric] !== undefined)
      .sort((a, b) =>
        isLowerBetter
          ? a.metrics[metric] - b.metrics[metric]
          : b.metrics[metric] - a.metrics[metric]
      );

    if (withMetric.length === 0) {
      return { content: [{ type: "text", text: `No experiments have the metric "${metric}".` }] };
    }

    const top = withMetric.slice(0, n);
    const lines = top.map((e, i) =>
      `${i + 1}. ${e.name} (${e.model}) — ${metric}: ${e.metrics[metric]}`
    );

    return {
      content: [{
        type: "text",
        text: `Top ${top.length} by ${metric} (${isLowerBetter ? "lower is better" : "higher is better"}):\n\n${lines.join("\n")}`,
      }],
    };
  },
);

// --- Resources ---

server.resource(
  "all-experiments",
  "experiments://all",
  async () => ({
    contents: [{
      uri: "experiments://all",
      text: JSON.stringify(Array.from(experiments.values()), null, 2),
    }],
  }),
);

server.resource(
  "experiment-summary",
  "experiments://summary",
  async () => {
    const all = Array.from(experiments.values());
    const byStatus = Object.groupBy(all, (e) => e.status);
    const completed = all.filter((e) => e.status === "completed");
    const avgAccuracy = completed.length > 0
      ? completed.reduce((sum, e) => sum + (e.metrics.accuracy ?? 0), 0) / completed.length
      : null;

    const summary = {
      total: all.length,
      byStatus: Object.fromEntries(
        Object.entries(byStatus).map(([status, exps]) => [status, exps?.length ?? 0])
      ),
      averageAccuracy: avgAccuracy?.toFixed(4) ?? "N/A",
      models: [...new Set(all.map((e) => e.model))],
    };

    return {
      contents: [{
        uri: "experiments://summary",
        text: JSON.stringify(summary, null, 2),
      }],
    };
  },
);

// --- Prompts ---

server.prompt(
  "analyze-results",
  "Generate a prompt to analyze all experiment results",
  {},
  () => {
    const all = Array.from(experiments.values());
    const data = JSON.stringify(all, null, 2);

    return {
      messages: [{
        role: "user" as const,
        content: {
          type: "text" as const,
          text: `Here are my ML experiment results:\n\n${data}\n\nPlease analyze these results and:\n1. Identify the best-performing experiment and why\n2. Suggest what might have gone wrong with failed experiments\n3. Recommend next experiments to try based on patterns you see\n4. Note any hyperparameter patterns that correlate with better results`,
        },
      }],
    };
  },
);

server.prompt(
  "debug-experiment",
  "Generate a prompt to debug a failed experiment",
  { id: z.string().describe("Experiment ID to debug") },
  ({ id }) => {
    const exp = experiments.get(id);
    if (!exp) {
      return {
        messages: [{
          role: "user" as const,
          content: { type: "text" as const, text: `Experiment "${id}" not found.` },
        }],
      };
    }

    return {
      messages: [{
        role: "user" as const,
        content: {
          type: "text" as const,
          text: `My ML experiment "${exp.name}" has ${exp.status === "failed" ? "failed" : `status: ${exp.status}`}.\n\nDetails:\n- Model: ${exp.model}\n- Hyperparameters: ${JSON.stringify(exp.hyperparameters)}\n- Metrics: ${JSON.stringify(exp.metrics)}\n- Notes: ${exp.notes}\n\nPlease help me debug this:\n1. What might have caused the poor results?\n2. What hyperparameter changes would you suggest?\n3. Are there any red flags in the metrics?\n4. What additional diagnostics should I run?`,
        },
      }],
    };
  },
);

// --- Start ---

console.error("Starting experiment-tracker MCP server...");
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Testing

```bash
npx @modelcontextprotocol/inspector npx tsx src/server.ts
```

Try these in the inspector:
1. `list_experiments` with no filters
2. `list_experiments` with `{ "status": "completed" }`
3. `get_experiment` with one of the IDs from the list
4. `create_experiment` with `{ "name": "New Test", "model": "llama-2", "notes": "Testing" }`
5. `list_experiments` again to verify the new experiment appears
6. `best_experiment` with `{ "metric": "accuracy" }`
7. `compare_experiments` with two experiment IDs

### Using with Claude Desktop

This is a practical server you can actually use. Add it to Claude Desktop and ask things like:

- "What experiments have I run?"
- "Compare my BERT and DistilBERT experiments"
- "Which experiment had the best accuracy?"
- "Create a new experiment for fine-tuning Llama 2 with learning rate 1e-4"
- "The DistilBERT NER experiment failed — help me debug it"

</details>
