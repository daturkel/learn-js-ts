# Module 13: Capstone Project

Apply everything you've learned by building a complete project.

**Prerequisites:** All previous modules
**Time:** 3-5 sessions

## Goal

Build a real, functional project that combines multiple skills from the curriculum. Choose one of the project ideas below (or come up with your own) and build it from scratch.

## What You Should Use

Your project should incorporate at least 4-5 of these:

- **TypeScript** — Type-safe code with interfaces and generics
- **Async patterns** — Promises, streaming, async/await
- **Node.js APIs** — File system, environment config, child processes
- **Testing** — At least a few Vitest tests for core logic
- **CLI or Web interface** — Commander.js CLI or Next.js web app
- **API integration** — REST API (Hono) or AI SDK calls
- **MCP** — Expose functionality as an MCP server (optional but recommended)

## Project Ideas

See [project-ideas.md](./project-ideas.md) for four detailed project specifications, ranging from CLI tools to full-stack web apps.

## Process

### 1. Plan (Session 1)

- Choose a project
- List the features you'll build (start small, expand later)
- Sketch the file structure
- Identify which modules/skills you'll use

### 2. Build core (Sessions 2-3)

- Set up the project (TypeScript, dependencies)
- Build the core functionality first (no UI)
- Write tests for the core logic
- Get the happy path working end-to-end

### 3. Add interface (Sessions 3-4)

- Add the CLI or web UI
- Connect it to your core logic
- Handle errors and edge cases

### 4. Polish (Session 5)

- Add an MCP server interface (if applicable)
- Clean up code and types
- Add missing error handling
- Make sure tests pass
- Write a README for your project

## Evaluation Criteria

There's no formal evaluation — this is self-paced learning. But here's what "good" looks like:

**Code quality:**
- TypeScript types used throughout (no `any`)
- Clean error handling (no silent failures)
- Reasonable file organization

**Functionality:**
- Core feature works end-to-end
- Handles common error cases (bad input, network failures)
- At least 5-10 meaningful tests

**Completeness:**
- Has a README explaining what it does and how to run it
- `package.json` has proper scripts (`dev`, `build`, `test`)
- Can be run by someone else following the README

## Tips

- **Start simpler than you think.** Get the minimum working, then add features.
- **Use `tsx` for development.** Don't worry about building/bundling until the end.
- **Copy patterns from exercises.** The exercises are reference implementations — reuse patterns you've practiced.
- **Test early.** Writing tests alongside your code catches bugs before they compound.
- **Use AI tools.** You're learning to be an AI engineer — use Claude, Cursor, etc. to help build your project. That's the whole point.
