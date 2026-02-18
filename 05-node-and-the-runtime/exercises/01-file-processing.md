# Exercise: File Processing

Read JSON files, compute aggregate statistics, and write a summary CSV.

## Goal

Practice Node.js file I/O by processing a directory of ML experiment results — the same kind of task you'd do with `pathlib` and `json` in Python.

## Setup

Create a project directory with some sample data:

```bash
mkdir -p file-exercise/data
cd file-exercise
npm init -y
npm install -D typescript tsx
```

Create these sample JSON files in `data/`:

**`data/exp-001.json`:**
```json
{
  "name": "bert-base-v1",
  "model": "bert-base",
  "accuracy": 0.891,
  "f1": 0.885,
  "trainingHours": 2.5,
  "epoch": 3,
  "date": "2024-01-15"
}
```

**`data/exp-002.json`:**
```json
{
  "name": "bert-large-v1",
  "model": "bert-large",
  "accuracy": 0.923,
  "f1": 0.918,
  "trainingHours": 8.0,
  "epoch": 5,
  "date": "2024-01-16"
}
```

**`data/exp-003.json`:**
```json
{
  "name": "gpt2-v1",
  "model": "gpt2",
  "accuracy": 0.845,
  "f1": 0.831,
  "trainingHours": 1.5,
  "epoch": 2,
  "date": "2024-01-17"
}
```

**`data/exp-004.json`:**
```json
{
  "name": "roberta-v1",
  "model": "roberta",
  "accuracy": 0.935,
  "f1": 0.930,
  "trainingHours": 6.0,
  "epoch": 4,
  "date": "2024-01-18"
}
```

**`data/notes.txt`:** (not JSON — your code should skip this)
```
These are experiment notes.
```

## Tasks

Create `process.ts`:

### 1. Read all JSON files from `data/`

- List files in the directory
- Filter to only `.json` files
- Read and parse each one
- Define a TypeScript interface for the experiment data

### 2. Compute aggregate statistics

- Total experiments
- Average accuracy
- Best model (highest accuracy)
- Total training hours
- Accuracy range (min to max)

### 3. Write a summary CSV

Write `summary.csv` with columns: `name, model, accuracy, f1, trainingHours`

Also print the aggregate stats to the console.

<details>
<summary>Hint: listing and filtering files</summary>

```typescript
import { readdir } from "node:fs/promises";

const files = await readdir("data");
const jsonFiles = files.filter(f => f.endsWith(".json"));
```
</details>

<details>
<summary>Solution</summary>

```typescript
import { readdir, readFile, writeFile } from "node:fs/promises";
import path from "node:path";

// --- Types ---

interface Experiment {
  name: string;
  model: string;
  accuracy: number;
  f1: number;
  trainingHours: number;
  epoch: number;
  date: string;
}

// --- Step 1: Read all JSON files ---

const dataDir = "data";
const files = await readdir(dataDir);
const jsonFiles = files.filter(f => f.endsWith(".json"));

const experiments: Experiment[] = [];
for (const file of jsonFiles) {
  const filePath = path.join(dataDir, file);
  const text = await readFile(filePath, "utf-8");
  const exp = JSON.parse(text) as Experiment;
  experiments.push(exp);
}

console.log(`Loaded ${experiments.length} experiments from ${jsonFiles.length} files`);

// --- Step 2: Compute stats ---

const avgAccuracy = experiments.reduce((sum, e) => sum + e.accuracy, 0) / experiments.length;
const totalHours = experiments.reduce((sum, e) => sum + e.trainingHours, 0);
const best = experiments.reduce((b, e) => e.accuracy > b.accuracy ? e : b);
const accuracies = experiments.map(e => e.accuracy);
const minAcc = Math.min(...accuracies);
const maxAcc = Math.max(...accuracies);

console.log(`\nAggregate Statistics:`);
console.log(`  Total experiments: ${experiments.length}`);
console.log(`  Average accuracy:  ${avgAccuracy.toFixed(3)}`);
console.log(`  Best model:        ${best.name} (${best.accuracy})`);
console.log(`  Total training:    ${totalHours.toFixed(1)} hours`);
console.log(`  Accuracy range:    ${minAcc.toFixed(3)} - ${maxAcc.toFixed(3)}`);

// --- Step 3: Write CSV ---

const header = "name,model,accuracy,f1,trainingHours";
const rows = experiments.map(e =>
  `${e.name},${e.model},${e.accuracy},${e.f1},${e.trainingHours}`
);
const csv = [header, ...rows].join("\n") + "\n";

await writeFile("summary.csv", csv);
console.log(`\nWrote summary.csv`);
```

### Python equivalent for comparison

```python
from pathlib import Path
import json
import csv

data_dir = Path("data")
experiments = []
for f in data_dir.glob("*.json"):
    experiments.append(json.loads(f.read_text()))

avg_accuracy = sum(e["accuracy"] for e in experiments) / len(experiments)
best = max(experiments, key=lambda e: e["accuracy"])

with open("summary.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "model", "accuracy", "f1", "trainingHours"])
    writer.writeheader()
    writer.writerows(experiments)
```

Key differences:
- `readdir` returns filenames, not `Path` objects — you join paths manually
- No `glob()` built in — filter with `.endsWith()` or use a npm package
- CSV writing is manual string building (or use an npm package like `csv-stringify`)
- Everything is `async` — every file operation needs `await`
</details>
