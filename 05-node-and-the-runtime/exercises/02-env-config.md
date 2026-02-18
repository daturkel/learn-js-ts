# Exercise: Environment Config Loader

Build a typed configuration loader with multiple sources.

## Goal

Build a config loader that reads from environment variables, a `.env` file, and a JSON config file — with proper priority, defaults, and validation. This is a common pattern in Node.js applications.

## Config Priority (highest to lowest)

1. Environment variables (`process.env`)
2. `.env` file (if it exists)
3. `config.json` (if it exists)
4. Default values

## Setup

Create a project directory:

```bash
mkdir config-exercise && cd config-exercise
npm init -y
npm install -D typescript tsx
```

Create a sample `.env` file:

```
PORT=8080
LOG_LEVEL=debug
```

Create a sample `config.json`:

```json
{
  "port": 3000,
  "host": "localhost",
  "logLevel": "info",
  "maxRetries": 5
}
```

## Your Task

Create `config.ts` that:

1. Defines a `Config` interface:
   - `port`: number (default: 3000)
   - `host`: string (default: "localhost")
   - `logLevel`: `"debug" | "info" | "warn" | "error"` (default: "info")
   - `maxRetries`: number (default: 3)
   - `apiKey`: string (required — no default, must be provided somewhere)

2. Implements `loadConfig(): Promise<Config>` that:
   - Reads `config.json` (silently ignores if missing)
   - Reads `.env` file and parses key=value lines (silently ignores if missing)
   - Checks `process.env` for overrides
   - Applies defaults for missing optional values
   - Throws if `apiKey` is missing from all sources

3. Test it by running with and without env vars:
   ```bash
   npx tsx config.ts
   API_KEY=sk-test npx tsx config.ts
   ```

<details>
<summary>Hint: Parsing .env files</summary>

A simple `.env` parser:
```typescript
function parseEnvFile(content: string): Record<string, string> {
  const result: Record<string, string> = {};
  for (const line of content.split("\n")) {
    const trimmed = line.trim();
    if (!trimmed || trimmed.startsWith("#")) continue;
    const eqIndex = trimmed.indexOf("=");
    if (eqIndex === -1) continue;
    const key = trimmed.slice(0, eqIndex).trim();
    const value = trimmed.slice(eqIndex + 1).trim();
    result[key] = value;
  }
  return result;
}
```
</details>

<details>
<summary>Hint: Reading files that might not exist</summary>

```typescript
async function readFileOrNull(path: string): Promise<string | null> {
  try {
    return await readFile(path, "utf-8");
  } catch {
    return null;
  }
}
```
</details>

<details>
<summary>Solution</summary>

```typescript
import { readFile } from "node:fs/promises";

// --- Types ---

type LogLevel = "debug" | "info" | "warn" | "error";

interface Config {
  port: number;
  host: string;
  logLevel: LogLevel;
  maxRetries: number;
  apiKey: string;
}

// --- Helpers ---

async function readFileOrNull(filePath: string): Promise<string | null> {
  try {
    return await readFile(filePath, "utf-8");
  } catch {
    return null;
  }
}

function parseEnvFile(content: string): Record<string, string> {
  const result: Record<string, string> = {};
  for (const line of content.split("\n")) {
    const trimmed = line.trim();
    if (!trimmed || trimmed.startsWith("#")) continue;
    const eqIndex = trimmed.indexOf("=");
    if (eqIndex === -1) continue;
    result[trimmed.slice(0, eqIndex).trim()] = trimmed.slice(eqIndex + 1).trim();
  }
  return result;
}

const validLogLevels: LogLevel[] = ["debug", "info", "warn", "error"];

function isValidLogLevel(value: string): value is LogLevel {
  return validLogLevels.includes(value as LogLevel);
}

// --- Main loader ---

async function loadConfig(): Promise<Config> {
  // Layer 1: Defaults
  const config: Partial<Config> = {
    port: 3000,
    host: "localhost",
    logLevel: "info",
    maxRetries: 3,
  };

  // Layer 2: config.json
  const jsonContent = await readFileOrNull("config.json");
  if (jsonContent) {
    try {
      const json = JSON.parse(jsonContent);
      if (typeof json.port === "number") config.port = json.port;
      if (typeof json.host === "string") config.host = json.host;
      if (typeof json.logLevel === "string" && isValidLogLevel(json.logLevel)) {
        config.logLevel = json.logLevel;
      }
      if (typeof json.maxRetries === "number") config.maxRetries = json.maxRetries;
      if (typeof json.apiKey === "string") config.apiKey = json.apiKey;
      console.log("  Loaded config.json");
    } catch (e) {
      console.warn("  Warning: config.json exists but is invalid JSON");
    }
  }

  // Layer 3: .env file
  const envContent = await readFileOrNull(".env");
  if (envContent) {
    const envVars = parseEnvFile(envContent);
    if (envVars.PORT) config.port = parseInt(envVars.PORT);
    if (envVars.HOST) config.host = envVars.HOST;
    if (envVars.LOG_LEVEL && isValidLogLevel(envVars.LOG_LEVEL)) {
      config.logLevel = envVars.LOG_LEVEL;
    }
    if (envVars.MAX_RETRIES) config.maxRetries = parseInt(envVars.MAX_RETRIES);
    if (envVars.API_KEY) config.apiKey = envVars.API_KEY;
    console.log("  Loaded .env file");
  }

  // Layer 4: process.env (highest priority)
  if (process.env.PORT) config.port = parseInt(process.env.PORT);
  if (process.env.HOST) config.host = process.env.HOST;
  if (process.env.LOG_LEVEL && isValidLogLevel(process.env.LOG_LEVEL)) {
    config.logLevel = process.env.LOG_LEVEL;
  }
  if (process.env.MAX_RETRIES) config.maxRetries = parseInt(process.env.MAX_RETRIES);
  if (process.env.API_KEY) config.apiKey = process.env.API_KEY;

  // Validate required fields
  if (!config.apiKey) {
    throw new Error(
      "API_KEY is required. Set it via environment variable, .env file, or config.json"
    );
  }

  return config as Config;
}

// --- Run ---

console.log("Loading config...");
try {
  const config = await loadConfig();
  console.log("\nFinal config:");
  console.log(JSON.stringify(config, null, 2));
} catch (error) {
  console.error(`\nError: ${(error as Error).message}`);
  process.exit(1);
}
```

Run it:
```bash
npx tsx config.ts                        # Fails — no API_KEY
API_KEY=sk-test npx tsx config.ts        # Works — env var provides API_KEY
API_KEY=sk-test PORT=9999 npx tsx config.ts  # Env var overrides .env and config.json
```
</details>
