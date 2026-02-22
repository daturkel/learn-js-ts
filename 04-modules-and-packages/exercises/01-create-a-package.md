# Exercise: Create a TypeScript Package

Initialize a TypeScript project from scratch and build a working module.

## Goal

Set up a proper TypeScript project with `npm`, write a module with exports, and run it. This is the equivalent of setting up a Python project with `pyproject.toml`.

## Tasks

### Step 1: Initialize the project

```bash
mkdir stats-lib && cd stats-lib
npm init -y
```

This creates `package.json` (like `pyproject.toml`).

Edit `package.json` to add `"type": "module"` (enables ES module `import`/`export` syntax):

```json
{
  "name": "stats-lib",
  "version": "1.0.0",
  "type": "module"
}
```

### Step 2: Install TypeScript

```bash
npm install -D typescript tsx
```

`-D` installs as dev dependency (like `pip install --dev`). These are tools needed for development, not runtime.

### Step 3: Configure TypeScript

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "declaration": true
  },
  "include": ["src"]
}
```

Key settings:
- `outDir: "dist"` — compiled JS goes to `dist/` (like Python's `build/`)
- `strict: true` — enables all strict type checks
- `declaration: true` — generates `.d.ts` type declaration files

### Step 4: Write the stats module

Create `src/stats.ts` with these functions:

```typescript
// Implement these:
export function mean(numbers: number[]): number { /* ... */ }
export function median(numbers: number[]): number { /* ... */ }
export function stddev(numbers: number[]): number { /* ... */ }
export function percentile(numbers: number[], p: number): number { /* ... */ }
```

### Step 5: Write the main entry point

Create `src/index.ts` that imports from your module and runs some examples:

```typescript
import { mean, median, stddev, percentile } from "./stats.js";  // Note: .js extension!

const data = [23, 45, 12, 67, 34, 89, 21, 56, 78, 43];
console.log(`Mean: ${mean(data)}`);
console.log(`Median: ${median(data)}`);
console.log(`Std Dev: ${stddev(data).toFixed(2)}`);
console.log(`95th percentile: ${percentile(data, 95)}`);
```

**Important:** In Node.js ESM, you must use `.js` extensions in imports even for `.ts` files. TypeScript compiles `.ts` to `.js`, so the import path must match the output.

### Step 6: Add npm scripts

Add to `package.json`:

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "tsx src/index.ts"
  }
}
```

- `npm run build` — compiles TS to JS (like building a Python package)
- `npm start` — runs the compiled JS
- `npm run dev` — runs TS directly (for development, like `python script.py`)

### Step 7: Build and run

```bash
npm run dev          # Run directly (development)
npm run build        # Compile to JS
npm start            # Run compiled output
```

<details>
<summary>Hint: median implementation</summary>

Sort a copy of the array, then pick the middle element:
```typescript
const sorted = [...numbers].sort((a, b) => a - b);
const mid = Math.floor(sorted.length / 2);
```
For even-length arrays, average the two middle elements.
</details>

<details>
<summary>Solution: src/stats.ts</summary>

```typescript
export function mean(numbers: number[]): number {
  if (numbers.length === 0) throw new Error("Cannot compute mean of empty array");
  return numbers.reduce((sum, n) => sum + n, 0) / numbers.length;
}

export function median(numbers: number[]): number {
  if (numbers.length === 0) throw new Error("Cannot compute median of empty array");
  const sorted = [...numbers].sort((a, b) => a - b);
  const mid = Math.floor(sorted.length / 2);

  if (sorted.length % 2 === 0) {
    return (sorted[mid - 1] + sorted[mid]) / 2;
  }
  return sorted[mid];
}

export function stddev(numbers: number[]): number {
  if (numbers.length === 0) throw new Error("Cannot compute stddev of empty array");
  const avg = mean(numbers);
  const squaredDiffs = numbers.map(n => (n - avg) ** 2);
  return Math.sqrt(mean(squaredDiffs));
}

export function percentile(numbers: number[], p: number): number {
  if (numbers.length === 0) throw new Error("Cannot compute percentile of empty array");
  if (p < 0 || p > 100) throw new Error("Percentile must be between 0 and 100");
  const sorted = [...numbers].sort((a, b) => a - b);
  const index = (p / 100) * (sorted.length - 1);
  const lower = Math.floor(index);
  const upper = Math.ceil(index);

  if (lower === upper) return sorted[lower];
  const weight = index - lower;
  return sorted[lower] * (1 - weight) + sorted[upper] * weight;
}
```
</details>

<details>
<summary>Solution: src/index.ts</summary>

```typescript
import { mean, median, stddev, percentile } from "./stats.js";

const data = [23, 45, 12, 67, 34, 89, 21, 56, 78, 43];

console.log("Data:", data);
console.log(`Mean: ${mean(data).toFixed(2)}`);
console.log(`Median: ${median(data)}`);
console.log(`Std Dev: ${stddev(data).toFixed(2)}`);
console.log(`95th percentile: ${percentile(data, 95).toFixed(2)}`);
console.log(`50th percentile: ${percentile(data, 50).toFixed(2)}`);
```
</details>

### Final directory structure

```
stats-lib/
├── package.json
├── tsconfig.json
├── node_modules/        ← git-ignored
├── src/
│   ├── stats.ts
│   └── index.ts
└── dist/                ← generated by npm run build
    ├── stats.js
    ├── stats.d.ts
    ├── index.js
    └── index.d.ts
```
