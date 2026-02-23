# Exercise: First Test Suite

Write tests for a configuration loader using Vitest.

## Goal

Practice writing tests with `describe`, `it`, `expect`, and matchers. The module under test is a simplified version of the config loader from Module 05.

## Setup

```bash
mkdir test-exercise && cd test-exercise
npm init -y
npm install -D typescript tsx vitest
```

Add to `package.json`:
```json
{ "type": "module", "scripts": { "test": "vitest run" } }
```

## The Code Under Test

Create `config.ts`:

```typescript
type LogLevel = "debug" | "info" | "warn" | "error";

export interface Config {
  port: number;
  host: string;
  logLevel: LogLevel;
  apiKey: string;
}

const validLogLevels: LogLevel[] = ["debug", "info", "warn", "error"];

export function parseConfig(json: string): Config {
  let raw: Record<string, unknown>;
  try {
    raw = JSON.parse(json);
  } catch {
    throw new Error("Invalid JSON");
  }

  if (typeof raw !== "object" || raw === null || Array.isArray(raw)) {
    throw new Error("Config must be an object");
  }

  const port = raw.port ?? 3000;
  if (typeof port !== "number" || port < 1 || port > 65535) {
    throw new Error("port must be a number between 1 and 65535");
  }

  const host = raw.host ?? "localhost";
  if (typeof host !== "string") {
    throw new Error("host must be a string");
  }

  const logLevel = (raw.logLevel ?? "info") as string;
  if (!validLogLevels.includes(logLevel as LogLevel)) {
    throw new Error(`logLevel must be one of: ${validLogLevels.join(", ")}`);
  }

  const apiKey = raw.apiKey;
  if (typeof apiKey !== "string" || apiKey.length === 0) {
    throw new Error("apiKey is required");
  }

  return { port: port as number, host, logLevel: logLevel as LogLevel, apiKey };
}

export function mergeWithEnv(config: Config, env: Record<string, string | undefined>): Config {
  return {
    ...config,
    port: env.PORT ? parseInt(env.PORT) : config.port,
    host: env.HOST ?? config.host,
    logLevel: (env.LOG_LEVEL as LogLevel) ?? config.logLevel,
    apiKey: env.API_KEY ?? config.apiKey,
  };
}
```

## Tasks

Create `config.test.ts` with tests covering:

### parseConfig tests
1. Valid config with all fields
2. Valid config with defaults (only apiKey provided)
3. Invalid JSON string
4. Missing apiKey (should throw)
5. Invalid port (negative, too high, string)
6. Invalid logLevel
7. Array instead of object
8. Null input to JSON.parse

### mergeWithEnv tests
1. Environment variables override config values
2. Missing env vars keep original values
3. PORT is parsed as integer

Run with: `npm test`

<details>
<summary>Hint: Testing that functions throw</summary>

```typescript
// Wrap the call in a function:
expect(() => parseConfig("not json")).toThrow("Invalid JSON");

// For partial message matching:
expect(() => parseConfig('{"port": -1}')).toThrow(/port/);
```
</details>

<details>
<summary>Solution</summary>

```typescript
import { describe, it, expect } from "vitest";
import { parseConfig, mergeWithEnv, Config } from "./config.js";

describe("parseConfig", () => {
  it("parses valid config with all fields", () => {
    const config = parseConfig(JSON.stringify({
      port: 8080,
      host: "0.0.0.0",
      logLevel: "debug",
      apiKey: "sk-test-123",
    }));

    expect(config).toEqual({
      port: 8080,
      host: "0.0.0.0",
      logLevel: "debug",
      apiKey: "sk-test-123",
    });
  });

  it("applies defaults when optional fields are missing", () => {
    const config = parseConfig('{"apiKey": "sk-test"}');

    expect(config.port).toBe(3000);
    expect(config.host).toBe("localhost");
    expect(config.logLevel).toBe("info");
    expect(config.apiKey).toBe("sk-test");
  });

  it("throws on invalid JSON", () => {
    expect(() => parseConfig("not json")).toThrow("Invalid JSON");
    expect(() => parseConfig("{missing: quotes}")).toThrow("Invalid JSON");
  });

  it("throws when apiKey is missing", () => {
    expect(() => parseConfig('{"port": 3000}')).toThrow("apiKey is required");
  });

  it("throws when apiKey is empty string", () => {
    expect(() => parseConfig('{"apiKey": ""}')).toThrow("apiKey is required");
  });

  it("throws on invalid port values", () => {
    expect(() => parseConfig('{"apiKey": "sk", "port": -1}')).toThrow("port");
    expect(() => parseConfig('{"apiKey": "sk", "port": 99999}')).toThrow("port");
    expect(() => parseConfig('{"apiKey": "sk", "port": "abc"}')).toThrow("port");
  });

  it("throws on invalid logLevel", () => {
    expect(() => parseConfig('{"apiKey": "sk", "logLevel": "verbose"}'))
      .toThrow("logLevel must be one of");
  });

  it("throws when input is an array", () => {
    expect(() => parseConfig("[1, 2, 3]")).toThrow("Config must be an object");
  });

  it("accepts valid log levels", () => {
    for (const level of ["debug", "info", "warn", "error"]) {
      const config = parseConfig(JSON.stringify({ apiKey: "sk", logLevel: level }));
      expect(config.logLevel).toBe(level);
    }
  });
});

describe("mergeWithEnv", () => {
  const baseConfig: Config = {
    port: 3000,
    host: "localhost",
    logLevel: "info",
    apiKey: "sk-base",
  };

  it("overrides values from environment", () => {
    const merged = mergeWithEnv(baseConfig, {
      PORT: "8080",
      HOST: "0.0.0.0",
      LOG_LEVEL: "debug",
      API_KEY: "sk-env",
    });

    expect(merged.port).toBe(8080);
    expect(merged.host).toBe("0.0.0.0");
    expect(merged.logLevel).toBe("debug");
    expect(merged.apiKey).toBe("sk-env");
  });

  it("keeps original values when env vars are missing", () => {
    const merged = mergeWithEnv(baseConfig, {});

    expect(merged).toEqual(baseConfig);
  });

  it("parses PORT as integer", () => {
    const merged = mergeWithEnv(baseConfig, { PORT: "9999" });
    expect(merged.port).toBe(9999);
    expect(typeof merged.port).toBe("number");
  });

  it("does not modify the original config", () => {
    const original = { ...baseConfig };
    mergeWithEnv(baseConfig, { PORT: "8080" });
    expect(baseConfig).toEqual(original);
  });
});
```

Run with `npm test` and you should see all tests passing.
</details>

---

**Next:** [Exercise 2: Mocking APIs →](/06-testing/exercises/02-mocking-apis.md)
