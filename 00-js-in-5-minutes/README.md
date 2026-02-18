# Module 00: JavaScript in 5 Minutes

A quick orientation before you write any code. Read this, then do the two exercises.

## What Is JavaScript?

JavaScript is the scripting language of the web. Every browser has a built-in JS engine. But JS hasn't been browser-only for over a decade -- **Node.js** (2009) lets you run JavaScript on the server, as a CLI tool, or anywhere Python runs.

You'll primarily use **Node.js** in this curriculum. Two newer alternatives exist:

| Runtime | Notes |
|---------|-------|
| **Node.js** | The standard. Like CPython for Python. What tutorials and libraries assume. |
| **Bun** | Newer, faster, has built-in TypeScript support. Growing fast. |
| **Deno** | Created by Node's original author. Different security model. Smaller ecosystem. |

All three run the same JavaScript. Think of them like CPython vs PyPy -- same language, different engines and standard libraries.

## The Ecosystem: A Python Translation

| Python | JavaScript | Notes |
|--------|-----------|-------|
| `python3` | `node` | The runtime |
| `pip` | `npm` | Package manager (bundled with Node) |
| `pyproject.toml` | `package.json` | Project metadata and dependencies |
| `venv/` or `.venv/` | `node_modules/` | Installed dependencies (always per-project, no global env) |
| `pipx run` | `npx` | Run a package without installing it globally |
| PyPI | npm registry | Package repository (over 2 million packages) |

One key difference: `node_modules` is *always* local to the project. There's no equivalent of installing packages into a global Python environment. Every project gets its own `node_modules/` automatically -- it's as if you always use a venv.

## JavaScript vs TypeScript

**TypeScript** is JavaScript with static types added on top. It compiles down to plain JavaScript before running.

```
TypeScript (.ts) --> compiler (tsc) --> JavaScript (.js) --> Node.js runs it
```

If you use Python type hints with mypy or pyright, this will feel familiar -- except TypeScript's type checking is not optional in a TS project. The compiler won't emit JavaScript if there are type errors (by default).

- **JavaScript** = the language every runtime actually executes
- **TypeScript** = a superset that adds types, then compiles to JS

This curriculum starts with plain JavaScript (Modules 00-01) so you see the raw language, then switches to TypeScript from Module 02 onward.

## The Runtime Model

Python and JavaScript are both single-threaded, but they handle concurrency differently:

- **Python**: Single-threaded due to the GIL. Uses `asyncio` for async I/O, or `threading`/`multiprocessing` for parallelism. You explicitly choose your concurrency model.
- **JavaScript**: Single-threaded with an **event loop** built in. All I/O is non-blocking by default. There's no `asyncio.run()` -- the event loop is always running.

The practical effect: in Python, you can write blocking code and it "just works" (slowly). In JavaScript, the ecosystem assumes everything is async. You'll spend Module 03 getting comfortable with this, but for now just know: **async is the default, not the exception**.

## Ecosystem Map

The JS ecosystem is large and moves fast. Here are the major categories you'll encounter:

| Category | Tools | Python Equivalent |
|----------|-------|-------------------|
| **Runtimes** | Node.js, Bun, Deno | CPython, PyPy |
| **Package managers** | npm, yarn, pnpm | pip, poetry, uv |
| **Linters** | ESLint | ruff, flake8 |
| **Formatters** | Prettier | black |
| **Test runners** | Vitest, Jest | pytest |
| **Bundlers** | esbuild, Vite, webpack | (no direct equivalent -- Python doesn't need bundling) |
| **Web frameworks** | Hono, Express, Next.js | FastAPI, Flask, Django |
| **Type checkers** | TypeScript compiler (tsc) | mypy, pyright |

You don't need to learn all of these. The curriculum introduces them as needed.

## What You'll Build

By the end of this curriculum, you'll have built:

- CLI tools with argument parsing and interactive prompts
- REST APIs with typed request/response handling
- A React/Next.js web application
- AI-powered tools using Anthropic and OpenAI SDKs
- An MCP (Model Context Protocol) server

Each builds on the previous modules. The early modules (00-03) are about learning the language; the later ones (07-13) are about building real things with it.

## Next Steps

1. Do [Exercise 01: REPL Exploration](./exercises/01-repl-exploration.md) (~20 minutes)
2. Do [Exercise 02: Python to JS Rosetta Stone](./exercises/02-python-to-js-rosetta.md) (~30 minutes)
3. Move on to [Module 01: Language Fundamentals](../01-language-fundamentals/README.md)
