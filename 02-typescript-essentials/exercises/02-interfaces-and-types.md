# Exercise: Interfaces and Types

Define TypeScript types for an ML experiment tracker.

## Goal

Practice defining interfaces, type aliases, union types, and optional properties by creating a type system for ML experiments.

## Setup

Make sure you've installed TypeScript and tsx (see [Setup](/SETUP.md)).

Create a new file called `experiment-types.ts`.

- To **type-check** without running: `tsc --strict --noEmit --target es2015 experiment-types.ts`
- To **run** the file and see output: `tsx experiment-types.ts`

## Requirements

### Step 1: Define the core types

Define interfaces/types for:

1. **`MetricValue`** — a single metric measurement:
   - `name`: string (e.g., "accuracy", "loss")
   - `value`: number
   - `step`: number (training step)
   - `timestamp`: Date (optional)

2. **`HyperparameterConfig`** — training hyperparameters:
   - `learningRate`: number
   - `batchSize`: number
   - `epochs`: number
   - `optimizer`: one of `"adam"`, `"sgd"`, `"adamw"` (use a union type)
   - `schedulerType`: string (optional)
   - Should also allow arbitrary additional params via an index signature

3. **`RunStatus`** — a type alias for: `"pending"` | `"running"` | `"completed"` | `"failed"`

4. **`Run`** — a single training run:
   - `id`: string
   - `status`: RunStatus
   - `hyperparameters`: HyperparameterConfig
   - `metrics`: array of MetricValue
   - `startedAt`: Date
   - `completedAt`: Date (optional)
   - `errorMessage`: string (optional, only present when status is "failed")

5. **`Experiment`** — groups multiple runs:
   - `id`: string
   - `name`: string
   - `description`: string (optional)
   - `model`: string
   - `dataset`: string
   - `runs`: array of Run
   - `tags`: array of string
   - `createdAt`: Date

### Step 2: Write typed functions

Write these functions with full type annotations:

1. `getCompletedRuns(experiment: Experiment): Run[]` — returns only completed runs
2. `getBestMetric(run: Run, metricName: string): MetricValue | undefined` — finds the highest value for a given metric across all steps
3. `summarizeExperiment(experiment: Experiment): ExperimentSummary` — returns a summary object (define the `ExperimentSummary` type yourself)

### Step 3: Create test data and verify

Create a sample experiment with 2-3 runs and call each function. Make sure TypeScript catches type errors — try passing wrong types intentionally.

<details>
<summary>Hint: Index signature for extra params</summary>

```typescript
interface HyperparameterConfig {
  learningRate: number;
  // ... other known fields ...
  [key: string]: unknown;  // Allows arbitrary additional properties
}
```

This is like Python's `TypedDict` with `total=False` extra keys, or using `**kwargs`.
</details>

<details>
<summary>Hint: Union type for optimizer</summary>

```typescript
type Optimizer = "adam" | "sgd" | "adamw";
```

This is like Python's `Literal["adam", "sgd", "adamw"]`.
</details>

<details>
<summary>Solution</summary>

```typescript
// --- Type Definitions ---

interface MetricValue {
  name: string;
  value: number;
  step: number;
  timestamp?: Date;
}

type Optimizer = "adam" | "sgd" | "adamw";

interface HyperparameterConfig {
  learningRate: number;
  batchSize: number;
  epochs: number;
  optimizer: Optimizer;
  schedulerType?: string;
  [key: string]: unknown;  // Allow extra params
}

type RunStatus = "pending" | "running" | "completed" | "failed";

interface Run {
  id: string;
  status: RunStatus;
  hyperparameters: HyperparameterConfig;
  metrics: MetricValue[];
  startedAt: Date;
  completedAt?: Date;
  errorMessage?: string;
}

interface Experiment {
  id: string;
  name: string;
  description?: string;
  model: string;
  dataset: string;
  runs: Run[];
  tags: string[];
  createdAt: Date;
}

interface ExperimentSummary {
  name: string;
  model: string;
  totalRuns: number;
  completedRuns: number;
  failedRuns: number;
  bestAccuracy: number | null;
  tags: string[];
}

// --- Functions ---

function getCompletedRuns(experiment: Experiment): Run[] {
  return experiment.runs.filter(r => r.status === "completed");
}

function getBestMetric(run: Run, metricName: string): MetricValue | undefined {
  const matching = run.metrics.filter(m => m.name === metricName);
  if (matching.length === 0) return undefined;
  return matching.reduce((best, m) => m.value > best.value ? m : best);
}

function summarizeExperiment(experiment: Experiment): ExperimentSummary {
  const completed = getCompletedRuns(experiment);
  const failed = experiment.runs.filter(r => r.status === "failed");

  let bestAccuracy: number | null = null;
  for (const run of completed) {
    const best = getBestMetric(run, "accuracy");
    if (best && (bestAccuracy === null || best.value > bestAccuracy)) {
      bestAccuracy = best.value;
    }
  }

  return {
    name: experiment.name,
    model: experiment.model,
    totalRuns: experiment.runs.length,
    completedRuns: completed.length,
    failedRuns: failed.length,
    bestAccuracy,
    tags: experiment.tags,
  };
}

// --- Test Data ---

const experiment: Experiment = {
  id: "exp-001",
  name: "BERT Sentiment Analysis",
  model: "bert-base-uncased",
  dataset: "imdb-reviews",
  tags: ["nlp", "sentiment", "bert"],
  createdAt: new Date("2024-01-15"),
  runs: [
    {
      id: "run-1",
      status: "completed",
      hyperparameters: {
        learningRate: 3e-5,
        batchSize: 32,
        epochs: 3,
        optimizer: "adam",
        warmupSteps: 100,     // Extra param — allowed by index signature
      },
      metrics: [
        { name: "accuracy", value: 0.87, step: 100 },
        { name: "accuracy", value: 0.91, step: 200 },
        { name: "loss", value: 0.32, step: 200 },
      ],
      startedAt: new Date("2024-01-15T10:00:00"),
      completedAt: new Date("2024-01-15T12:30:00"),
    },
    {
      id: "run-2",
      status: "completed",
      hyperparameters: {
        learningRate: 5e-5,
        batchSize: 16,
        epochs: 5,
        optimizer: "adamw",
      },
      metrics: [
        { name: "accuracy", value: 0.89, step: 100 },
        { name: "accuracy", value: 0.93, step: 300 },
        { name: "loss", value: 0.25, step: 300 },
      ],
      startedAt: new Date("2024-01-16T09:00:00"),
      completedAt: new Date("2024-01-16T14:00:00"),
    },
    {
      id: "run-3",
      status: "failed",
      hyperparameters: {
        learningRate: 1e-3,
        batchSize: 64,
        epochs: 10,
        optimizer: "sgd",
      },
      metrics: [
        { name: "loss", value: 5.2, step: 10 },
      ],
      startedAt: new Date("2024-01-17T08:00:00"),
      errorMessage: "CUDA out of memory",
    },
  ],
};

// Test
console.log("Completed runs:", getCompletedRuns(experiment).length);  // 2
console.log("Best accuracy run-2:", getBestMetric(experiment.runs[1], "accuracy"));
// { name: "accuracy", value: 0.93, step: 300 }
console.log("Summary:", summarizeExperiment(experiment));

// Type error examples (uncomment to see TS catch them):
// experiment.runs[0].status = "unknown";  // Error: not assignable to RunStatus
// experiment.runs[0].hyperparameters.optimizer = "momentum";  // Error: not in Optimizer
// getBestMetric(experiment, "accuracy");  // Error: Experiment is not a Run
```

### Python Equivalent

```python
from dataclasses import dataclass, field
from typing import Literal, TypedDict
from datetime import datetime

class MetricValue(TypedDict):
    name: str
    value: float
    step: int
    timestamp: datetime | None

Optimizer = Literal["adam", "sgd", "adamw"]
RunStatus = Literal["pending", "running", "completed", "failed"]

# In Python you'd use Pydantic or dataclasses.
# TypeScript interfaces have NO runtime presence — they're compile-time only.
# That's the biggest conceptual difference from Python's runtime type checking.
```
</details>
