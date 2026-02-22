# Exercise: Type Narrowing and Unknown

Practice writing type-safe code that handles data of unknown shape — the pattern you'll use constantly when working with API responses, parsed JSON, and external data.

## Setup

Create `type-narrowing.ts` and run with `npx tsx type-narrowing.ts`.

## Background

When you receive data from outside your program — a parsed JSON string, an API response, user input — TypeScript can't know its shape. There are two ways to handle this:

- **`any`** — opts out of type checking entirely. The data can be used as anything with no errors, but also no safety.
- **`unknown`** — forces you to check the shape before using it. Safer, but requires explicit narrowing.

```typescript
const data: any = JSON.parse(someString);
data.foo.bar.baz;  // No error — TypeScript trusts you. Runtime error if wrong.

const safe: unknown = JSON.parse(someString);
safe.foo;  // Error: Object is of type 'unknown'
if (typeof safe === "object" && safe !== null && "foo" in safe) {
  // Now TypeScript knows safe has a foo property
}
```

## Tasks

### Task 1: Narrow `unknown` manually

Write a function `parseAge(value: unknown): number` that:
- Returns the value if it's a number
- Parses and returns it if it's a numeric string (e.g. `"42"`)
- Throws a `TypeError` with a descriptive message otherwise

```typescript
parseAge(25)       // 25
parseAge("25")     // 25
parseAge("hello")  // throws TypeError
parseAge(null)     // throws TypeError
```

<details>
<summary>Hint</summary>

Use `typeof` to narrow:
```typescript
if (typeof value === "number") { ... }
if (typeof value === "string") { ... }
```

For the string case, `Number(value)` converts it — but `Number("hello")` returns `NaN`. Check with `isNaN()`.
</details>

### Task 2: Type guard functions

A **type guard** is a function that narrows a type using `value is T` as its return type:

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}
```

When you call `isString(x)` in an `if`, TypeScript knows `x` is a `string` inside the block.

Write these type guards:

```typescript
function isString(value: unknown): value is string
function isNumber(value: unknown): value is number
function isRecord(value: unknown): value is Record<string, unknown>
function hasKey<K extends string>(obj: Record<string, unknown>, key: K): obj is Record<K, unknown>
```

Test them:
```typescript
const data: unknown = { model: "bert", accuracy: 0.91 };

if (isRecord(data) && hasKey(data, "model") && isString(data.model)) {
  console.log(data.model.toUpperCase());  // TypeScript knows this is a string
}
```

<details>
<summary>Hint: hasKey</summary>

```typescript
function hasKey<K extends string>(
  obj: Record<string, unknown>,
  key: K
): obj is Record<K, unknown> {
  return key in obj;
}
```

The `K extends string` constraint means the key must be a string literal type, so TypeScript can track the specific key name.
</details>

### Task 3: Parse an API response

You're calling an AI API that returns JSON. The response shape depends on whether it succeeded:

```typescript
// Success:
// { "ok": true, "model": "claude-3", "content": "Hello!", "tokens": 42 }

// Error:
// { "ok": false, "error": "rate_limit_exceeded", "retryAfter": 30 }
```

Define these types:

```typescript
interface ApiSuccess {
  ok: true;
  model: string;
  content: string;
  tokens: number;
}

interface ApiError {
  ok: false;
  error: string;
  retryAfter?: number;
}

type ApiResponse = ApiSuccess | ApiError;
```

Then write `parseApiResponse(raw: unknown): ApiResponse` that validates the raw data and returns a typed `ApiResponse`, throwing a descriptive error if the shape is wrong.

Test it:

```typescript
const good = parseApiResponse({
  ok: true, model: "claude-3", content: "Hello!", tokens: 42
});
if (good.ok) {
  console.log(good.content, good.tokens);  // TypeScript knows these exist
}

const bad = parseApiResponse({
  ok: false, error: "rate_limit_exceeded", retryAfter: 30
});
if (!bad.ok) {
  console.log(bad.error, bad.retryAfter);  // TypeScript knows these exist
}

parseApiResponse({ ok: true });             // throws — missing fields
parseApiResponse("not an object");          // throws
parseApiResponse({ ok: "yes" });            // throws — ok must be boolean
```

<details>
<summary>Hint: structure</summary>

```typescript
function parseApiResponse(raw: unknown): ApiResponse {
  if (!isRecord(raw)) throw new TypeError("Expected an object");
  if (!hasKey(raw, "ok") || typeof raw.ok !== "boolean") {
    throw new TypeError("Missing or invalid field: ok");
  }
  if (raw.ok === true) {
    // validate ApiSuccess fields
  } else {
    // validate ApiError fields
  }
}
```
</details>

### Task 4: Discriminated union narrowing

Write `describeResponse(response: ApiResponse): string` that returns a human-readable summary. TypeScript should narrow the type automatically based on `response.ok` — no casts needed.

```typescript
describeResponse({ ok: true, model: "claude-3", content: "Hi!", tokens: 42 })
// "claude-3 responded with 42 tokens: Hi!"

describeResponse({ ok: false, error: "rate_limit_exceeded", retryAfter: 30 })
// "Request failed: rate_limit_exceeded (retry after 30s)"

describeResponse({ ok: false, error: "invalid_key" })
// "Request failed: invalid_key"
```

<details>
<summary>Solution</summary>

```typescript
// --- Task 1 ---

function parseAge(value: unknown): number {
  if (typeof value === "number") return value;
  if (typeof value === "string") {
    const n = Number(value);
    if (!isNaN(n)) return n;
  }
  throw new TypeError(`Cannot parse age from ${typeof value}: ${value}`);
}

console.log(parseAge(25));      // 25
console.log(parseAge("25"));    // 25
try { parseAge("hello"); } catch (e) { console.log((e as Error).message); }
try { parseAge(null); } catch (e) { console.log((e as Error).message); }

// --- Task 2 ---

function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}

function hasKey<K extends string>(
  obj: Record<string, unknown>,
  key: K
): obj is Record<K, unknown> {
  return key in obj;
}

const data: unknown = { model: "bert", accuracy: 0.91 };
if (isRecord(data) && hasKey(data, "model") && isString(data.model)) {
  console.log(data.model.toUpperCase());  // "BERT"
}

// --- Task 3 ---

interface ApiSuccess {
  ok: true;
  model: string;
  content: string;
  tokens: number;
}

interface ApiError {
  ok: false;
  error: string;
  retryAfter?: number;
}

type ApiResponse = ApiSuccess | ApiError;

function parseApiResponse(raw: unknown): ApiResponse {
  if (!isRecord(raw)) throw new TypeError("Expected an object");
  if (!hasKey(raw, "ok") || typeof raw.ok !== "boolean") {
    throw new TypeError("Missing or invalid field: ok");
  }

  if (raw.ok === true) {
    if (!hasKey(raw, "model") || !isString(raw.model))
      throw new TypeError("Missing or invalid field: model");
    if (!hasKey(raw, "content") || !isString(raw.content))
      throw new TypeError("Missing or invalid field: content");
    if (!hasKey(raw, "tokens") || !isNumber(raw.tokens))
      throw new TypeError("Missing or invalid field: tokens");
    return { ok: true, model: raw.model, content: raw.content, tokens: raw.tokens };
  } else {
    if (!hasKey(raw, "error") || !isString(raw.error))
      throw new TypeError("Missing or invalid field: error");
    const retryAfter = hasKey(raw, "retryAfter") && isNumber(raw.retryAfter)
      ? raw.retryAfter
      : undefined;
    return { ok: false, error: raw.error, retryAfter };
  }
}

const good = parseApiResponse({ ok: true, model: "claude-3", content: "Hello!", tokens: 42 });
if (good.ok) console.log(good.content, good.tokens);

const bad = parseApiResponse({ ok: false, error: "rate_limit_exceeded", retryAfter: 30 });
if (!bad.ok) console.log(bad.error, bad.retryAfter);

try { parseApiResponse({ ok: true }); } catch (e) { console.log((e as Error).message); }
try { parseApiResponse("not an object"); } catch (e) { console.log((e as Error).message); }

// --- Task 4 ---

function describeResponse(response: ApiResponse): string {
  if (response.ok) {
    return `${response.model} responded with ${response.tokens} tokens: ${response.content}`;
  } else {
    const retry = response.retryAfter !== undefined ? ` (retry after ${response.retryAfter}s)` : "";
    return `Request failed: ${response.error}${retry}`;
  }
}

console.log(describeResponse({ ok: true, model: "claude-3", content: "Hi!", tokens: 42 }));
console.log(describeResponse({ ok: false, error: "rate_limit_exceeded", retryAfter: 30 }));
console.log(describeResponse({ ok: false, error: "invalid_key" }));
```

**Key observations:**

- `unknown` forces you to prove the shape before using the data. `any` skips that proof — and skips the safety.
- Type guards (`value is T`) teach TypeScript what you've verified so it can narrow downstream.
- Discriminated unions (`ok: true` / `ok: false`) let TypeScript narrow automatically in `if/else` — no casts, no runtime checks beyond the initial parse.
- This pattern (`parse at the boundary, use typed values everywhere inside`) is the standard approach for handling external data in TypeScript.
</details>
