# Module 05: Node.js and the Runtime

Node.js standard library and runtime APIs, mapped from Python equivalents.

**Time:** 1-2 sessions

## Key Difference: Node's stdlib is Thin

Python's stdlib is "batteries included" — `json`, `os`, `pathlib`, `subprocess`, `http`, `csv`, etc. Node's stdlib is minimal. The community fills gaps with npm packages.

This is a cultural choice, not a limitation. Where Python gives you one obvious way, Node gives you a small core and lets the ecosystem compete.

## File System: `node:fs/promises`

```python
# Python
from pathlib import Path
import json

data = Path("config.json").read_text()
config = json.loads(data)

Path("output.txt").write_text("hello")

files = list(Path(".").iterdir())
Path("new_dir").mkdir(exist_ok=True)
```

```typescript
// TypeScript
import { readFile, writeFile, readdir, mkdir } from "node:fs/promises";

const data = await readFile("config.json", "utf-8");
const config = JSON.parse(data);

await writeFile("output.txt", "hello");

const files = await readdir(".");
await mkdir("new_dir", { recursive: true });  // recursive = exist_ok
```

Always import from `node:fs/promises` (async). There's also `node:fs` with sync methods (`readFileSync`), but those block the event loop.

Common operations:

| Python | Node.js | Notes |
|--------|---------|-------|
| `Path(f).read_text()` | `readFile(f, "utf-8")` | Must specify encoding |
| `Path(f).write_text(s)` | `writeFile(f, s)` | |
| `Path(d).iterdir()` | `readdir(d)` | Returns filenames, not paths |
| `Path(d).mkdir(exist_ok=True)` | `mkdir(d, { recursive: true })` | |
| `Path(f).stat()` | `stat(f)` | Returns size, dates, etc. |
| `Path(f).exists()` | `access(f).then(() => true).catch(() => false)` | Awkward — no simple `.exists()` |
| `Path(f).unlink()` | `rm(f)` or `unlink(f)` | |
| `shutil.rmtree(d)` | `rm(d, { recursive: true })` | |

### Reading JSON files

```typescript
import { readFile } from "node:fs/promises";

// Common pattern for JSON config files
async function loadJson<T>(path: string): Promise<T> {
  const text = await readFile(path, "utf-8");
  return JSON.parse(text) as T;
}

const config = await loadJson<{ port: number; host: string }>("config.json");
```

## Path: `node:path`

```python
# Python
from pathlib import Path

full = Path("data") / "experiments" / "results.json"
parent = full.parent
name = full.name          # "results.json"
stem = full.stem          # "results"
ext = full.suffix         # ".json"
absolute = full.resolve()
```

```typescript
// TypeScript
import path from "node:path";

const full = path.join("data", "experiments", "results.json");
const parent = path.dirname(full);
const name = path.basename(full);        // "results.json"
const stem = path.basename(full, ".json"); // "results"
const ext = path.extname(full);          // ".json"
const absolute = path.resolve(full);
```

### Current file location

```python
# Python
import os
this_dir = os.path.dirname(__file__)
```

```typescript
// TypeScript (ES modules)
const thisDir = import.meta.dirname;     // Node 21+
// Or for older Node:
// import { fileURLToPath } from "node:url";
// const thisDir = path.dirname(fileURLToPath(import.meta.url));
```

## Process: `node:process`

```typescript
import process from "node:process";

// Environment variables (like os.environ)
const apiKey = process.env.API_KEY;          // string | undefined
const port = parseInt(process.env.PORT ?? "3000");

// Command-line arguments (like sys.argv)
const args = process.argv;
// args[0] = path to node
// args[1] = path to script
// args[2+] = user arguments
const userArgs = process.argv.slice(2);

// Current directory (like os.getcwd())
const cwd = process.cwd();

// Exit (like sys.exit())
process.exit(1);

// Standard I/O
process.stdout.write("no newline");
process.stderr.write("error output");
```

### Environment variables and .env files

Node has no built-in `.env` file loading (unlike some Python frameworks). Starting in Node 20, there's `--env-file`:

```bash
node --env-file=.env src/index.js
```

For older Node, use the `dotenv` package:

```bash
npm install dotenv
```

```typescript
import "dotenv/config";  // Loads .env file into process.env
console.log(process.env.API_KEY);
```

## HTTP with fetch()

`fetch` is built into Node 18+ (no package needed). It's like Python's `httpx`:

```typescript
// GET request
const response = await fetch("https://api.example.com/data");
if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}
const data = await response.json();

// POST request with JSON body
const response = await fetch("https://api.example.com/data", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Alice", score: 95 }),
});

// With auth header
const response = await fetch(url, {
  headers: { Authorization: `Bearer ${token}` },
});
```

**Critical difference:** `fetch` does NOT throw on 4xx/5xx responses. You must check `response.ok`:

```typescript
// Python httpx — throws on error status
response = httpx.get(url)
response.raise_for_status()  # Throws on 4xx/5xx

// JavaScript fetch — does NOT throw
const response = await fetch(url);
// response.status might be 404, but no exception!
if (!response.ok) {
  throw new Error(`HTTP ${response.status}`);
}
```

## Child Processes: `node:child_process`

```python
# Python
import subprocess

# Simple — capture output
result = subprocess.run(["ls", "-la"], capture_output=True, text=True)
print(result.stdout)

# Stream output
proc = subprocess.Popen(["tail", "-f", "log.txt"], stdout=subprocess.PIPE)
for line in proc.stdout:
    print(line)
```

```typescript
// TypeScript
import { execSync, exec, spawn } from "node:child_process";

// Simple — synchronous (blocks event loop, fine for scripts)
const output = execSync("ls -la", { encoding: "utf-8" });
console.log(output);

// Async with callback
exec("ls -la", (error, stdout, stderr) => {
  if (error) throw error;
  console.log(stdout);
});

// Stream output (like subprocess.Popen)
const proc = spawn("tail", ["-f", "log.txt"]);
proc.stdout.on("data", (chunk: Buffer) => {
  console.log(chunk.toString());
});
proc.on("close", (code: number) => {
  console.log(`Exited with code ${code}`);
});
```

For modern async usage, you can promisify `exec`:

```typescript
import { exec } from "node:child_process";
import { promisify } from "node:util";

const execAsync = promisify(exec);
const { stdout } = await execAsync("git log --oneline -5");
console.log(stdout);
```

## Further Reading

- [Node.js API Documentation](https://nodejs.org/docs/latest/api/) — Complete reference
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices) — Production patterns

## Exercises

1. [File Processing](./exercises/01-file-processing.md) — Read JSON files, compute stats, write CSV
2. [Env Config](./exercises/02-env-config.md) — Build a typed configuration loader
3. [Child Processes](./exercises/03-child-processes.md) — Analyze git log output

---

**Next:** [Exercise 1: File Processing →](/05-node-and-the-runtime/exercises/01-file-processing.md)
