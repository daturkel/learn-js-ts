# Module 06: Testing

Testing in JavaScript/TypeScript with Vitest, compared to Python's pytest.


## Vitest — The Modern Test Runner

Vitest is to JS what pytest is to Python — the modern default. It's fast, supports TypeScript natively, and has a Jest-compatible API.

```bash
npm install -D vitest
```

Add to `package.json`:
```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run"
  }
}
```

- `npm test` — runs in watch mode (re-runs on file changes)
- `npm run test:run` — runs once and exits (like `pytest`)

## Test Structure

Side by side with pytest:

```python
# Python — test_math.py
import pytest

def test_addition():
    assert 1 + 1 == 2

def test_division_by_zero():
    with pytest.raises(ZeroDivisionError):
        1 / 0

class TestStringMethods:
    def test_upper(self):
        assert "hello".upper() == "HELLO"

    def test_split(self):
        assert "a,b,c".split(",") == ["a", "b", "c"]
```

```typescript
// TypeScript — math.test.ts
import { describe, it, expect } from "vitest";

it("adds numbers", () => {
  expect(1 + 1).toBe(2);
});

it("throws on division by zero", () => {
  expect(() => divide(1, 0)).toThrow();
});

describe("string methods", () => {
  it("converts to upper case", () => {
    expect("hello".toUpperCase()).toBe("HELLO");
  });

  it("splits on delimiter", () => {
    expect("a,b,c".split(",")).toEqual(["a", "b", "c"]);
  });
});
```

Key differences:
- `describe` groups tests (like a pytest class)
- `it` or `test` defines a test (like `def test_...`)
- `expect(value).matcher()` instead of `assert`
- Arrow functions `() => { }` instead of `def`
- File naming: `*.test.ts` or `*.spec.ts` (Vitest finds these automatically)

## Matchers

| pytest | Vitest | What it checks |
|--------|--------|---------------|
| `assert x == y` | `expect(x).toBe(y)` | Strict equality (same reference or primitive) |
| `assert x == y` (objects) | `expect(x).toEqual(y)` | Deep equality (compares contents) |
| `assert x in collection` | `expect(collection).toContain(x)` | Contains element |
| `assert x is None` | `expect(x).toBeNull()` | Is null |
| `assert x is not None` | `expect(x).toBeDefined()` | Is not undefined |
| `assert x` | `expect(x).toBeTruthy()` | Is truthy |
| `assert not x` | `expect(x).toBeFalsy()` | Is falsy |
| `assert len(x) == 3` | `expect(x).toHaveLength(3)` | Collection length |
| `with pytest.raises(E):` | `expect(() => fn()).toThrow()` | Throws an error |
| `assert x > 5` | `expect(x).toBeGreaterThan(5)` | Numeric comparison |
| `assert "foo" in str` | `expect(str).toContain("foo")` | String contains |

### toBe vs toEqual

This is the most important distinction:

```typescript
// toBe — strict equality (===), checks reference for objects
expect(1).toBe(1);                       // Pass
expect({ a: 1 }).toBe({ a: 1 });        // FAIL — different objects!

// toEqual — deep equality, compares structure
expect({ a: 1 }).toEqual({ a: 1 });     // Pass
expect([1, 2]).toEqual([1, 2]);          // Pass
```

Use `toBe` for primitives, `toEqual` for objects and arrays.

### Partial matching

```typescript
expect({ name: "Alice", age: 30, email: "a@b.com" }).toMatchObject({
  name: "Alice",
  age: 30,
  // email doesn't need to match
});
```

## Async Testing

Test async functions by making the test function async:

```typescript
it("fetches data", async () => {
  const result = await fetchData();
  expect(result).toBeDefined();
  expect(result.status).toBe("ok");
});

it("rejects on invalid input", async () => {
  await expect(fetchData("bad")).rejects.toThrow("Invalid");
});
```

## Mocking

### vi.fn() — Mock Functions

Like Python's `MagicMock()`:

```python
# Python
from unittest.mock import MagicMock

callback = MagicMock()
callback(1, 2)
callback.assert_called_once_with(1, 2)
```

```typescript
// TypeScript
import { vi } from "vitest";

const callback = vi.fn();
callback(1, 2);
expect(callback).toHaveBeenCalledOnce();
expect(callback).toHaveBeenCalledWith(1, 2);

// Mock return values
const mockFetch = vi.fn().mockResolvedValue({ ok: true, json: () => ({ data: 42 }) });
```

### vi.mock() — Mock Modules

Like Python's `mock.patch()`:

```python
# Python
from unittest.mock import patch

with patch("mymodule.requests.get") as mock_get:
    mock_get.return_value.json.return_value = {"data": 42}
    result = my_function()
```

```typescript
// TypeScript
import { vi } from "vitest";

// Mock an entire module
vi.mock("node:fs/promises", () => ({
  readFile: vi.fn().mockResolvedValue('{"key": "value"}'),
}));

// Now any import of readFile uses the mock
import { readFile } from "node:fs/promises";
```

### vi.spyOn() — Spy on Methods

Like `mock.patch.object()`:

```typescript
const spy = vi.spyOn(console, "log");
myFunction();
expect(spy).toHaveBeenCalledWith("expected output");
spy.mockRestore();  // Restore original
```

## Setup and Teardown

```typescript
import { beforeEach, afterEach, beforeAll, afterAll } from "vitest";

beforeAll(() => {
  // Runs once before all tests (like pytest fixtures with session scope)
});

beforeEach(() => {
  // Runs before each test (like pytest fixtures with function scope)
});

afterEach(() => {
  // Runs after each test
  vi.restoreAllMocks();
});

afterAll(() => {
  // Runs once after all tests
});
```

## Code Coverage

```bash
npm install -D @vitest/coverage-v8
npx vitest run --coverage
```

## Configuration

Create `vitest.config.ts` (optional — Vitest works without config):

```typescript
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: true,      // No need to import describe/it/expect
    coverage: {
      reporter: ["text", "html"],
    },
  },
});
```

## Further Reading

- [Vitest Documentation](https://vitest.dev/) — Complete reference

## Exercises

1. [First Test Suite](./exercises/01-first-test-suite.md) — Test the config loader from Module 05
2. [Mocking APIs](./exercises/02-mocking-apis.md) — Mock fetch and test API functions

---

**Next:** [Exercise 1: First Test Suite →](/06-testing/exercises/01-first-test-suite.md)
