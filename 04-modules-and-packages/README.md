# Module 04: Modules and Packages

How JavaScript organizes code into modules and manages dependencies. If you've used `import` in Python and `pip install` in a terminal, the concepts are familiar -- the syntax and tooling are different.

## ESM vs CommonJS

JavaScript has **two** module systems. This is the single most confusing thing about the ecosystem for newcomers.

### ESM (ECMAScript Modules) -- the modern standard

```typescript
// Importing
import { readFile } from "node:fs/promises";
import express from "express";

// Exporting
export function add(a: number, b: number): number {
  return a + b;
}

export default class Calculator { /* ... */ }
```

ESM uses `import`/`export` syntax. It's the official JavaScript standard, works in browsers and Node.js, and is what you should always use in new code.

**Python comparison:**

| Python | TypeScript (ESM) |
|--------|-----------------|
| `from os import path` | `import { join } from "node:path"` |
| `import json` | `import * as fs from "node:fs"` |
| `from . import utils` | `import { helper } from "./utils.js"` |

### CommonJS (CJS) -- the legacy Node.js system

```javascript
// Importing
const fs = require("fs");
const { readFile } = require("fs/promises");

// Exporting
module.exports = { add, subtract };
module.exports = Calculator;
```

CommonJS uses `require()` and `module.exports`. You'll see this everywhere in older Node.js code, Stack Overflow answers, and many npm packages.

### Why both exist

Node.js was created in 2009. JavaScript had no module system at the time, so Node invented CommonJS. The official ESM standard wasn't finalized until 2015, and Node didn't fully support it until 2020. The ecosystem is still migrating.

**The rule: always use ESM (`import`/`export`) in new code.** You'll encounter CommonJS in existing projects and documentation, so you need to recognize it, but don't write it.

How Node.js decides which system to use:
- If `package.json` has `"type": "module"` -- files are ESM by default
- If `package.json` has `"type": "commonjs"` (or no `type` field) -- files are CJS by default
- `.mjs` files are always ESM, `.cjs` files are always CJS (regardless of package.json)

## package.json

The `package.json` file is the equivalent of Python's `pyproject.toml`. It defines your project's metadata, dependencies, and scripts.

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "type": "module",
  "main": "./dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "tsc --watch",
    "test": "vitest"
  },
  "dependencies": {
    "zod": "^3.22.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "vitest": "^1.0.0"
  }
}
```

### Key fields

| Field | Purpose | Python equivalent |
|-------|---------|-------------------|
| `name` | Package name | `[project] name` in pyproject.toml |
| `version` | Semver version | `[project] version` |
| `type` | `"module"` for ESM | No equivalent (Python always uses its import system) |
| `main` | Entry point for the package | Top-level `__init__.py` |
| `scripts` | Named commands | Makefile targets, or `[project.scripts]` |
| `dependencies` | Runtime deps | `[project.dependencies]` |
| `devDependencies` | Dev-only deps | `[project.optional-dependencies]` dev group |

### dependencies vs devDependencies

```json
{
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "vitest": "^1.0.0",
    "@types/express": "^4.17.0"
  }
}
```

- **`dependencies`** -- packages your code needs to run (like `express`, `zod`). Installed in production.
- **`devDependencies`** -- packages only needed during development (TypeScript compiler, test runner, type definitions). Not installed in production.

In Python terms: `dependencies` is your `requirements.txt`, `devDependencies` is your `requirements-dev.txt`.

### The scripts field

The `scripts` field is like a project-local Makefile. You run scripts with `npm run <name>`:

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "tsc --watch",
    "test": "vitest",
    "lint": "eslint src/",
    "format": "prettier --write src/"
  }
}
```

```bash
npm run build     # Compiles TypeScript
npm run test      # Runs tests
npm start         # Special shorthand: "start" doesn't need "run"
npm test          # Special shorthand: "test" doesn't need "run"
```

Scripts can reference any package installed in `node_modules/.bin/` without a full path. When you run `npm run build`, npm temporarily adds `node_modules/.bin/` to your PATH, so `tsc` resolves to the locally installed TypeScript compiler.

## node_modules

When you run `npm install`, packages are downloaded into a `node_modules/` directory in your project root. This is the equivalent of a Python virtual environment's `site-packages/`, but it's always project-local.

```
my-project/
  node_modules/        <-- installed packages (like .venv/lib/python3.x/site-packages/)
    typescript/
    vitest/
    zod/
    ...hundreds more
  package.json
  package-lock.json
  src/
```

### Why node_modules is huge

npm installs a **flat dependency tree**. If package A depends on package B which depends on package C, all three end up in `node_modules/`. A typical project can have hundreds or thousands of packages in `node_modules/`, easily reaching 200MB+.

This is normal. Don't panic. It's a cultural difference -- the JS ecosystem favors small, focused packages (sometimes controversially so).

### Always .gitignore it

Add `node_modules/` to `.gitignore`. Never commit it. Anyone cloning your project runs `npm install` to recreate it from `package.json`.

```gitignore
node_modules/
dist/
```

### package-lock.json

`package-lock.json` is an auto-generated file that pins the exact version of every installed package (including transitive dependencies). It's the equivalent of `poetry.lock` or `pip freeze` output.

- **Do** commit `package-lock.json` to version control
- **Don't** edit it manually
- It ensures everyone on your team gets identical dependency versions

## npm Commands

The npm CLI is bundled with Node.js. Here are the commands you'll use constantly:

### Project initialization

```bash
npm init                  # Interactive setup -- creates package.json
npm init -y               # Non-interactive -- accepts all defaults
```

Python equivalent: `poetry init` or manually creating `pyproject.toml`.

### Installing packages

```bash
npm install zod           # Install a runtime dependency
npm install -D vitest     # Install a dev dependency (-D = --save-dev)
npm install               # Install all deps from package.json
```

| npm | Python equivalent |
|-----|-------------------|
| `npm install zod` | `pip install zod` |
| `npm install -D vitest` | `pip install vitest` (then add to dev deps manually) |
| `npm install` | `pip install -r requirements.txt` |
| `npm uninstall zod` | `pip uninstall zod` |

### Running scripts

```bash
npm run build             # Run the "build" script
npm run test              # Run the "test" script
npm start                 # Shorthand for "npm run start"
npm test                  # Shorthand for "npm run test"
```

### Running packages without installing

```bash
npx tsc --init            # Run tsc without installing globally
npx create-next-app       # Run a package's CLI tool
```

`npx` downloads and runs a package temporarily. It's like `pipx run` -- you don't need to install globally. Use it for one-off commands and project scaffolding tools.

### Linking local packages

```bash
npm link                  # In the package you want to make available
npm link my-package       # In the project that wants to use it
```

This creates a symlink so you can develop and test local packages together without publishing them. Similar to `pip install -e .` (editable installs).

## Import Syntax Details

### Named exports vs default exports

This is a concept Python doesn't have. A module can export values in two ways:

**Named exports** -- you export specific names, and importers use those exact names:

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

// main.ts
import { add, multiply } from "./math.js";
```

**Default exports** -- the module exports one "main" thing, and the importer can name it anything:

```typescript
// calculator.ts
export default class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

// main.ts
import Calculator from "./calculator.js";
import Calc from "./calculator.js"; // This also works -- you pick the name
```

**A module can have both:**

```typescript
// api.ts
export default class ApiClient { /* ... */ }
export interface ApiConfig { /* ... */ }
export const DEFAULT_TIMEOUT = 5000;

// main.ts
import ApiClient, { ApiConfig, DEFAULT_TIMEOUT } from "./api.js";
```

**Convention:** Many style guides prefer named exports only, because default exports make renaming easy but refactoring harder (no consistent name to search for). You'll see both in the wild.

### Re-exports and barrel files

A common pattern is creating an `index.ts` file that re-exports from multiple modules, so consumers have a single import point:

```typescript
// utils/math.ts
export function add(a: number, b: number): number { return a + b; }

// utils/string.ts
export function capitalize(s: string): string {
  return s.charAt(0).toUpperCase() + s.slice(1);
}

// utils/index.ts  (the "barrel file")
export { add } from "./math.js";
export { capitalize } from "./string.js";

// main.ts -- import from the barrel
import { add, capitalize } from "./utils/index.js";
```

Python equivalent: an `__init__.py` that imports from submodules to create a clean public API.

### Importing built-in Node.js modules

Use the `node:` prefix for Node.js built-in modules:

```typescript
import { readFile } from "node:fs/promises";
import path from "node:path";
import { execSync } from "node:child_process";
```

The `node:` prefix is optional but recommended -- it makes clear you're importing a built-in, not an npm package. Python doesn't have this issue because the stdlib and PyPI don't share names (usually).

### Importing JSON

```typescript
// In Node.js with ESM, you need an import attribute:
import data from "./config.json" with { type: "json" };

// Or just read and parse it:
import { readFile } from "node:fs/promises";
const data = JSON.parse(await readFile("./config.json", "utf-8"));
```

The second approach (read + parse) is more common and doesn't require special configuration.

### Dynamic imports

Static `import` statements must be at the top of the file. If you need to import conditionally or lazily, use dynamic `import()`:

```typescript
// Dynamic import returns a Promise
const { default: chalk } = await import("chalk");

// Conditional import
if (process.env.NODE_ENV === "development") {
  const { setupDevTools } = await import("./dev-tools.js");
  setupDevTools();
}
```

Python equivalent: `importlib.import_module("module_name")` or just `import module` inside a function.

### File extensions in imports

One gotcha: when using ESM in TypeScript, you often need to use `.js` extensions in imports, even though your source files are `.ts`:

```typescript
// In src/main.ts, importing src/utils.ts:
import { helper } from "./utils.js";  // Yes, .js -- even though the file is .ts
```

This is because TypeScript doesn't rewrite import paths. The compiled output will be `.js` files, so the import needs to reference the `.js` file that will exist at runtime. Some configurations (like using a bundler) let you omit extensions, but this is the standard behavior with plain `tsc`.

## Version Ranges in package.json

npm uses semantic versioning (semver). The version strings in `package.json` use range syntax:

| Syntax | Meaning | Example |
|--------|---------|---------|
| `"^3.22.0"` | Compatible with 3.x.x (minor + patch updates) | 3.22.0 to <4.0.0 |
| `"~3.22.0"` | Patch updates only | 3.22.0 to <3.23.0 |
| `"3.22.0"` | Exact version | Only 3.22.0 |
| `">=3.22.0"` | At least this version | 3.22.0+ |

The `^` (caret) is the default and most common. It allows non-breaking updates.

## Further Reading

- [npm documentation](https://docs.npmjs.com/) -- the official reference
- [Node.js ESM documentation](https://nodejs.org/api/esm.html) -- how Node handles ESM
- [TypeScript module resolution](https://www.typescriptlang.org/docs/handbook/modules/theory.html) -- how TS finds modules

## Next Steps

1. Do [Exercise 01: Create a Package](./exercises/01-create-a-package.md) (~30 minutes)
2. Do [Exercise 02: Multi-Package Workspace](./exercises/02-publish-local-package.md) (~45 minutes)
3. Move on to [Module 05: Node.js and the Runtime](../05-node-and-the-runtime/README.md)
