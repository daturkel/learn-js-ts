# Module 12: Build Tools and Runtimes

Understand the JS/TS toolchain: TypeScript configuration, bundling, and alternative runtimes.


## Why This Module?

Throughout this curriculum, we've used `tsx` to run TypeScript directly. That works great for development, but there's a lot more to the toolchain. This module covers what happens "under the hood" and when you need to care about it.

In Python terms: this is like understanding `pyproject.toml`, build backends (setuptools, poetry), virtual environments, and alternative interpreters (CPython, PyPy) — the stuff you can ignore until you can't.

## TypeScript Configuration (tsconfig.json)

`tsconfig.json` controls how TypeScript compiles your code. It's the equivalent of Python's `pyproject.toml` for type checking settings.

### Essential Fields

```json
{
  "compilerOptions": {
    // Where compiled JS goes
    "outDir": "dist",
    "rootDir": "src",

    // Which JS version to target
    "target": "ES2022",

    // Module system
    "module": "Node16",
    "moduleResolution": "Node16",

    // Strictness
    "strict": true,
    "noUncheckedIndexedAccess": true,

    // Interop
    "esModuleInterop": true,
    "resolveJsonModule": true,

    // Output
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Key Settings Explained

**`target`** — What JS version to compile down to:
- `ES2022` — Modern Node.js (top-level await, private fields)
- `ES2020` — Good default for broad Node.js support
- `ES5` — Ancient browsers (you probably don't need this)

**`module`** — Module system:
- `Node16` / `NodeNext` — Use this for Node.js projects (handles ESM + CJS properly)
- `ESNext` — For bundler-based projects (Next.js, Vite)

**`strict`** — Enables all strict type checks. Always use this. It's like running `mypy --strict` in Python.

**`moduleResolution`** — How TS finds imported modules:
- `Node16` — Match the `module` setting for Node.js projects
- `Bundler` — For projects using a bundler (Vite, webpack, Next.js)

### Common Configs

For a **Node.js CLI/server**:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src"]
}
```

For a **Next.js app** (mostly auto-configured):
```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "jsx": "preserve",
    "strict": true,
    "paths": { "@/*": ["./*"] }
  }
}
```

## The Compilation Pipeline

```
TypeScript Source (.ts)
        │
        ▼
┌─ TypeScript Compiler (tsc) ─┐
│  1. Type checking            │
│  2. Transpile TS → JS        │
│  3. Generate .d.ts files     │
│  4. Generate source maps     │
└──────────────────────────────┘
        │
        ▼
JavaScript Output (.js + .d.ts + .js.map)
        │
        ▼
    Node.js runs the .js files
```

In development, `tsx` (or `ts-node`) skips the compilation step and runs TypeScript directly. For production or distribution, you compile to JavaScript.

```bash
# Development — run directly
npx tsx src/index.ts

# Production — compile then run
npx tsc                  # compile
node dist/index.js       # run compiled JS
```

## Bundling

Bundlers combine many files into one (or a few). This is mainly for:
- **Browser code** — browsers can't read `node_modules`
- **Distributable CLIs** — ship one file instead of requiring `npm install`
- **Edge functions** — platforms like Cloudflare Workers require a single file

### esbuild

The fastest bundler. Written in Go, extremely fast:

```bash
npm install -D esbuild
```

```bash
# Bundle everything into one file
npx esbuild src/index.ts --bundle --platform=node --outfile=dist/index.js

# With minification
npx esbuild src/index.ts --bundle --platform=node --minify --outfile=dist/index.js
```

esbuild handles TypeScript natively — no `tsc` needed. But it doesn't do type checking (it strips types without checking them).

### When to Bundle

| Scenario | Bundle? | Why |
|----------|---------|-----|
| Server/API | No | Node.js reads node_modules fine |
| CLI tool | Maybe | Faster startup, easier distribution |
| Browser code | Yes | Browsers need bundled code |
| Edge functions | Yes | Platforms require single file |
| Library/package | No | Let consumers bundle |

## Alternative Runtimes

### Bun

Bun is a drop-in Node.js replacement that's significantly faster. It includes:
- **Runtime** — runs JS/TS (no `tsx` needed)
- **Package manager** — `bun install` (faster than `npm`)
- **Bundler** — `bun build` (like esbuild)
- **Test runner** — `bun test` (like Vitest)

```bash
# Install Bun
curl -fsSL https://bun.sh/install | bash

# Run TypeScript directly
bun run src/index.ts

# Install packages (much faster than npm)
bun install

# Run tests
bun test

# Bundle
bun build src/index.ts --outfile=dist/index.js
```

Bun is compatible with most npm packages and Node.js APIs. Think of it as "Node.js but faster, with batteries included."

### Deno

Deno is another alternative runtime by the creator of Node.js. Key differences:
- TypeScript built-in (like Bun)
- Secure by default (explicit permissions for file/network access)
- URL imports (no `node_modules`)
- Standard library included

```bash
# Run TypeScript
deno run src/index.ts

# With permissions
deno run --allow-net --allow-read src/server.ts
```

Deno has improved Node.js compatibility significantly but isn't as drop-in as Bun. Use it if you want the security model or its standard library.

### Runtime Comparison

| Feature | Node.js | Bun | Deno |
|---------|---------|-----|------|
| TS support | Via tsx/ts-node | Built-in | Built-in |
| Package manager | npm/yarn/pnpm | bun (built-in) | deno add |
| Speed | Baseline | ~3x faster startup | ~2x faster startup |
| npm compat | N/A (it's the standard) | Very high | Good (improving) |
| Test runner | Via Vitest/Jest | Built-in | Built-in |
| Maturity | Very mature | Young but stable | Mature |

### Which to use?

- **Node.js** — Default choice. Maximum compatibility, largest ecosystem.
- **Bun** — When you want speed and convenience. Good for new projects.
- **Deno** — When you want security features or its standard library.

For this curriculum, Node.js is the default. Bun is worth trying in Exercise 02 to see the speed difference.

## Package Scripts

`package.json` scripts are how you define project commands:

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "lint": "eslint src/",
    "typecheck": "tsc --noEmit",
    "test": "vitest"
  }
}
```

```bash
npm run dev        # Development with hot reload
npm run build      # Compile TypeScript
npm start          # Run compiled code
npm test           # Run tests (special — no "run" needed)
npm run typecheck  # Type check without compiling
```

This is similar to Python's `[project.scripts]` in `pyproject.toml` or Makefile targets.

## Further Reading

- [TSConfig Reference](https://www.typescriptlang.org/tsconfig)
- [esbuild Documentation](https://esbuild.github.io/)
- [Bun Documentation](https://bun.sh/docs)
- [Deno Documentation](https://docs.deno.com/)

## Exercises

1. [TSConfig Deep Dive](./exercises/01-tsconfig-deep-dive.md) — Understand and configure TypeScript settings
2. [Try Bun](./exercises/02-try-bun.md) — Run your existing projects with Bun
3. [Bundle for Distribution](./exercises/03-bundle-for-distribution.md) — Bundle a CLI tool into a single file

---

**Next:** [Exercise 1: TSConfig Deep Dive →](/12-build-tools-and-runtimes/exercises/01-tsconfig-deep-dive.md)
