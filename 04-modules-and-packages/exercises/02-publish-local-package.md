# Exercise: Multi-Package Workspace

Create two linked packages using npm workspaces.

## Goal

Learn npm workspaces — the equivalent of Python monorepos with multiple packages that depend on each other. You'll create a `stats` library and a `cli` that uses it.

## Setup

### Step 1: Create the workspace root

```bash
mkdir my-workspace && cd my-workspace
```

Create a root `package.json`:

```json
{
  "name": "my-workspace",
  "private": true,
  "workspaces": ["packages/*"]
}
```

- `"private": true` — prevents accidentally publishing the root
- `"workspaces"` — tells npm where to find sub-packages

### Step 2: Create the stats package

```bash
mkdir -p packages/stats/src
```

Create `packages/stats/package.json`:

```json
{
  "name": "@learn/stats",
  "version": "1.0.0",
  "type": "module",
  "exports": "./dist/index.js",
  "scripts": {
    "build": "tsc"
  }
}
```

The `@learn/` prefix is a "scope" — like a namespace. It prevents name collisions on npm. Convention is `@org/package`.

Create `packages/stats/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "dist",
    "strict": true,
    "declaration": true
  },
  "include": ["src"]
}
```

### Step 3: Write the stats library

Create `packages/stats/src/index.ts`. Export at least `mean`, `median`, and `stddev` (you can reuse code from the previous exercise).

### Step 4: Create the CLI package

```bash
mkdir -p packages/cli/src
```

Create `packages/cli/package.json`:

```json
{
  "name": "@learn/cli",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "stats-cli": "dist/index.js"
  },
  "dependencies": {
    "@learn/stats": "1.0.0"
  },
  "scripts": {
    "build": "tsc",
    "dev": "tsx src/index.ts"
  }
}
```

The `"dependencies"` section references `@learn/stats` — npm workspaces will link it locally (no need to publish).

Create `packages/cli/tsconfig.json` (same as stats).

### Step 5: Write the CLI

Create `packages/cli/src/index.ts`:

```typescript
import { mean, median, stddev } from "@learn/stats";

const input = process.argv.slice(2).map(Number);

if (input.length === 0 || input.some(isNaN)) {
  console.log("Usage: stats-cli <number> <number> ...");
  console.log("Example: stats-cli 10 20 30 40 50");
  process.exit(1);
}

console.log(`Numbers: ${input.join(", ")}`);
console.log(`Mean:    ${mean(input).toFixed(2)}`);
console.log(`Median:  ${median(input)}`);
console.log(`Stddev:  ${stddev(input).toFixed(2)}`);
```

### Step 6: Install and build

From the workspace root:

```bash
npm install                          # Links workspaces
npm run build --workspace=packages/stats   # Build stats first
npm run build --workspace=packages/cli     # Then build cli
```

### Step 7: Test it

```bash
npx stats-cli 10 20 30 40 50
# Or:
npm run dev --workspace=packages/cli -- 10 20 30 40 50
```

<details>
<summary>Hint: If imports fail</summary>

Make sure:
1. You built the stats package first (`npm run build --workspace=packages/stats`)
2. The stats `package.json` has `"exports": "./dist/index.js"`
3. Both packages have `"type": "module"`
</details>

<details>
<summary>Solution: packages/stats/src/index.ts</summary>

```typescript
export function mean(numbers: number[]): number {
  if (numbers.length === 0) throw new Error("Empty array");
  return numbers.reduce((sum, n) => sum + n, 0) / numbers.length;
}

export function median(numbers: number[]): number {
  if (numbers.length === 0) throw new Error("Empty array");
  const sorted = [...numbers].sort((a, b) => a - b);
  const mid = Math.floor(sorted.length / 2);
  return sorted.length % 2 === 0
    ? (sorted[mid - 1] + sorted[mid]) / 2
    : sorted[mid];
}

export function stddev(numbers: number[]): number {
  if (numbers.length === 0) throw new Error("Empty array");
  const avg = mean(numbers);
  const squaredDiffs = numbers.map(n => (n - avg) ** 2);
  return Math.sqrt(mean(squaredDiffs));
}
```
</details>

### Final structure

```
my-workspace/
├── package.json                    ← root (private, defines workspaces)
├── node_modules/                   ← shared across packages
│   └── @learn/
│       └── stats -> ../../packages/stats   ← symlink!
├── packages/
│   ├── stats/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── src/index.ts
│   │   └── dist/                   ← built output
│   └── cli/
│       ├── package.json
│       ├── tsconfig.json
│       ├── src/index.ts
│       └── dist/
```

### Python comparison

This is like having a monorepo with:
```
my-workspace/
├── pyproject.toml              ← workspace/monorepo config
├── packages/
│   ├── stats/
│   │   ├── pyproject.toml
│   │   └── src/stats/...
│   └── cli/
│       ├── pyproject.toml      ← depends on stats
│       └── src/cli/...
```

Using `pip install -e .` for each package (editable installs) is the Python equivalent of npm's workspace symlinks.

---

**Next:** [Module 05: Node & the Runtime →](/05-node-and-the-runtime/README.md)
