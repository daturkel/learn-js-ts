# Exercise: Try Bun

Run your existing projects with Bun and compare the experience to Node.js.

## Setup

Install Bun:

```bash
curl -fsSL https://bun.sh/install | bash
```

Verify:
```bash
bun --version
```

## Tasks

### 1. Run TypeScript directly

Create a new file `bun-test.ts` (no project setup needed):

```typescript
const data = [1, 2, 3, 4, 5];
const doubled = data.map(x => x * 2);
console.log(`Doubled: ${doubled}`);

interface Config {
  name: string;
  debug: boolean;
}

const config: Config = { name: "test", debug: true };
console.log(`Config: ${JSON.stringify(config)}`);
```

Run it:
```bash
bun run bun-test.ts
```

No `tsx`, no `tsconfig.json`, no `package.json` needed. Bun runs TypeScript natively.

### 2. Compare `npm install` vs `bun install`

Go to one of your existing projects (e.g., the experiment API from Module 08):

```bash
# Delete node_modules to start fresh
rm -rf node_modules

# Time npm install
time npm install

# Delete again
rm -rf node_modules

# Time bun install
time bun install
```

Compare the times. Bun is typically 5-25x faster.

### 3. Run an existing project with Bun

Take your Hono server from Module 08 and run it with Bun:

```bash
bun run src/index.ts
```

Does it work? Most Node.js code works with Bun out of the box.

### 4. Use Bun's test runner

Create `math.ts`:

```typescript
export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}
```

Create `math.test.ts`:

```typescript
import { expect, test, describe } from "bun:test";
import { add, multiply } from "./math";

describe("math", () => {
  test("add", () => {
    expect(add(2, 3)).toBe(5);
    expect(add(-1, 1)).toBe(0);
  });

  test("multiply", () => {
    expect(multiply(3, 4)).toBe(12);
    expect(multiply(0, 100)).toBe(0);
  });
});
```

Run:
```bash
bun test
```

The API is nearly identical to Vitest/Jest. The main difference is `import { test } from "bun:test"` instead of `import { test } from "vitest"`.

### 5. Benchmark: Bun vs Node.js startup

Create `startup-bench.ts`:

```typescript
console.log(`Runtime: ${typeof Bun !== "undefined" ? "Bun" : "Node.js"}`);
console.log(`Time: ${performance.now().toFixed(2)}ms`);
```

Run with both runtimes:
```bash
time bun run startup-bench.ts
time npx tsx startup-bench.ts
```

Bun typically starts ~3x faster because it doesn't need to load `tsx` and can run TypeScript directly.

### 6. Bun as a bundler

```bash
bun build src/index.ts --outfile=dist/bundle.js --target=node
```

This is Bun's built-in bundler — similar to esbuild but integrated into the runtime.

<details>
<summary>Solution / Expected Results</summary>

### Expected install times

```
npm install:  ~5-15 seconds (varies by project)
bun install:  ~0.5-2 seconds
```

Bun uses a global cache and a faster resolution algorithm.

### Bun compatibility

Most Node.js code works in Bun. Common things that might not work:
- Some native addons (C++ extensions)
- Node.js-specific APIs that Bun hasn't implemented yet (rare)
- Some edge cases in `node:child_process`

For the projects in this curriculum, everything should work.

### When to use Bun vs Node.js

**Use Bun when:**
- Starting a new project (faster everything)
- You want built-in TypeScript without `tsx`
- Package installation speed matters (CI/CD, monorepos)
- You want a built-in test runner and bundler

**Stick with Node.js when:**
- Maximum compatibility needed
- Using native addons
- Production deployment where stability matters most
- Your team is already standardized on Node.js

### Bun cheatsheet

| Node.js | Bun |
|---------|-----|
| `npm install` | `bun install` |
| `npx tsx file.ts` | `bun run file.ts` |
| `npm test` (vitest) | `bun test` |
| `npx esbuild ...` | `bun build ...` |
| `npm init -y` | `bun init` |
| `npm run dev` | `bun run dev` |

</details>
