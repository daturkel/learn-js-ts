# Exercise: Generics

Implement generic types and functions in TypeScript.

## Goal

Practice generics by implementing reusable typed utilities. If you use Python type hints with `TypeVar` and `Generic`, these concepts will feel familiar — the syntax is just different.

## Setup

Create `generics.ts` and run with `npx tsx generics.ts`.

## Tasks

### Task 1: Result Type

Implement a `Result<T, E>` discriminated union — a type-safe way to represent success or failure without exceptions. This is similar to Rust's `Result` and a formalization of the `{ ok, config } | { ok, error }` pattern from Module 01.

```typescript
// Define these types:
type Ok<T> = { ok: true; value: T };
type Err<E> = { ok: false; error: E };
type Result<T, E = Error> = Ok<T> | Err<E>;

// Write helper functions:
function ok<T>(value: T): Ok<T> { /* ... */ }
function err<E>(error: E): Err<E> { /* ... */ }

// Write a function that uses Result:
function safeDivide(a: number, b: number): Result<number, string> { /* ... */ }

// Test:
const result = safeDivide(10, 3);
if (result.ok) {
  console.log(result.value);  // TypeScript KNOWS this is a number here
} else {
  console.log(result.error);  // TypeScript KNOWS this is a string here
}
```

### Task 2: Generic Functions

Implement these generic utility functions:

```typescript
// 1. first<T> — returns first element of array, or undefined
function first<T>(items: T[]): T | undefined { /* ... */ }

// 2. groupBy<T> — groups array items by a key function
function groupBy<T>(items: T[], keyFn: (item: T) => string): Record<string, T[]> { /* ... */ }

// 3. mapValues<V, R> — transforms all values in a Record
function mapValues<V, R>(obj: Record<string, V>, fn: (value: V) => R): Record<string, R> { /* ... */ }
```

Test with:

```typescript
// first
console.log(first([10, 20, 30]));    // 10
console.log(first([]));              // undefined
console.log(first(["a", "b"]));      // "a" — TypeScript infers T = string

// groupBy
const experiments = [
  { name: "bert-1", model: "bert" },
  { name: "bert-2", model: "bert" },
  { name: "gpt-1", model: "gpt" },
];
console.log(groupBy(experiments, e => e.model));
// { bert: [{name:"bert-1",...}, {name:"bert-2",...}], gpt: [{name:"gpt-1",...}] }

// mapValues
const counts = { bert: 2, gpt: 1, t5: 3 };
console.log(mapValues(counts, n => n * 100));
// { bert: 200, gpt: 100, t5: 300 }
```

### Task 3: Generic Constraint

Write a `maxBy` function with a constraint — it only works on arrays of items that have a numeric property:

```typescript
function maxBy<T>(items: T[], valueFn: (item: T) => number): T | undefined { /* ... */ }

// Should work:
const runs = [
  { id: "run-1", accuracy: 0.89 },
  { id: "run-2", accuracy: 0.93 },
  { id: "run-3", accuracy: 0.91 },
];
const best = maxBy(runs, r => r.accuracy);
console.log(best);  // { id: "run-2", accuracy: 0.93 }
```

<details>
<summary>Hint: groupBy</summary>

Build up the result object with reduce:
```typescript
items.reduce((groups, item) => {
  const key = keyFn(item);
  const existing = groups[key] ?? [];
  return { ...groups, [key]: [...existing, item] };
}, {} as Record<string, T[]>);
```

The `as Record<string, T[]>` tells TypeScript the initial `{}` has this type.
</details>

<details>
<summary>Hint: mapValues</summary>

Use `Object.entries()` to iterate, then `Object.fromEntries()` to rebuild:
```typescript
Object.fromEntries(
  Object.entries(obj).map(([key, value]) => [key, fn(value)])
)
```
</details>

<details>
<summary>Solution</summary>

```typescript
// --- Task 1: Result Type ---

type Ok<T> = { ok: true; value: T };
type Err<E> = { ok: false; error: E };
type Result<T, E = Error> = Ok<T> | Err<E>;

function ok<T>(value: T): Ok<T> {
  return { ok: true, value };
}

function err<E>(error: E): Err<E> {
  return { ok: false, error };
}

function safeDivide(a: number, b: number): Result<number, string> {
  if (b === 0) return err("Division by zero");
  return ok(a / b);
}

// Test Result
const r1 = safeDivide(10, 3);
if (r1.ok) {
  console.log("Result:", r1.value);   // 3.333...
} else {
  console.log("Error:", r1.error);
}

const r2 = safeDivide(10, 0);
if (r2.ok) {
  console.log("Result:", r2.value);
} else {
  console.log("Error:", r2.error);   // "Division by zero"
}

// --- Task 2: Generic Functions ---

function first<T>(items: T[]): T | undefined {
  return items[0];  // Returns undefined for empty arrays naturally
}

function groupBy<T>(items: T[], keyFn: (item: T) => string): Record<string, T[]> {
  return items.reduce((groups, item) => {
    const key = keyFn(item);
    const existing = groups[key] ?? [];
    return { ...groups, [key]: [...existing, item] };
  }, {} as Record<string, T[]>);
}

function mapValues<V, R>(
  obj: Record<string, V>,
  fn: (value: V) => R
): Record<string, R> {
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => [key, fn(value)])
  ) as Record<string, R>;
}

// Test first
console.log(first([10, 20, 30]));    // 10
console.log(first([]));              // undefined
console.log(first(["a", "b"]));      // "a"

// Test groupBy
const experiments = [
  { name: "bert-1", model: "bert" },
  { name: "bert-2", model: "bert" },
  { name: "gpt-1", model: "gpt" },
];
console.log(groupBy(experiments, e => e.model));

// Test mapValues
const counts = { bert: 2, gpt: 1, t5: 3 };
console.log(mapValues(counts, n => n * 100));

// --- Task 3: maxBy ---

function maxBy<T>(items: T[], valueFn: (item: T) => number): T | undefined {
  if (items.length === 0) return undefined;
  return items.reduce((best, item) =>
    valueFn(item) > valueFn(best) ? item : best
  );
}

const runs = [
  { id: "run-1", accuracy: 0.89 },
  { id: "run-2", accuracy: 0.93 },
  { id: "run-3", accuracy: 0.91 },
];
console.log(maxBy(runs, r => r.accuracy));  // { id: "run-2", accuracy: 0.93 }
console.log(maxBy([], (x: number) => x));   // undefined
```

### Python Comparison

```python
from typing import TypeVar, Callable

T = TypeVar("T")
V = TypeVar("V")
R = TypeVar("R")

def first(items: list[T]) -> T | None:
    return items[0] if items else None

def group_by(items: list[T], key_fn: Callable[[T], str]) -> dict[str, list[T]]:
    groups: dict[str, list[T]] = {}
    for item in items:
        key = key_fn(item)
        groups.setdefault(key, []).append(item)
    return groups

def map_values(obj: dict[str, V], fn: Callable[[V], R]) -> dict[str, R]:
    return {k: fn(v) for k, v in obj.items()}
```

The generic syntax is quite similar:
- Python: `T = TypeVar("T")` then `def fn(x: T) -> T`
- TypeScript: `function fn<T>(x: T): T` — type param is inline, no separate declaration
</details>

---

**Next:** [Exercise 4: Type Narrowing →](/02-typescript-essentials/exercises/04-type-narrowing.md)
