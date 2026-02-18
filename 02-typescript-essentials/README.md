# Module 02: TypeScript Essentials

TypeScript is JavaScript with a compile-time type system bolted on. If you already use Python type hints with mypy or pyright, the core idea is familiar -- but TypeScript's type system is considerably more powerful, and unlike Python, the compiler actually enforces it.

---

## Why TypeScript

JavaScript, like Python, is dynamically typed. You can assign a string to a variable and later reassign it to a number. TypeScript adds a static type layer that catches errors **before your code runs**.

**How it compares to Python type hints:**

| Aspect | Python type hints | TypeScript |
|---|---|---|
| Enforcement | Ignored at runtime; optional tools (mypy, pyright) check them | Enforced by the compiler; code won't compile if types are wrong |
| Adoption | Gradual -- you can add hints to some functions and skip others | Gradual -- you can use `any` to opt out, or mix `.js` and `.ts` files |
| Type erasure | Hints exist at runtime (`__annotations__`) but aren't checked | Types are completely erased at runtime; the output is plain JS |
| Ecosystem | Many libraries lack type stubs | Nearly all major JS libraries have type definitions (`@types/*` packages) |

**Structural vs. nominal typing:**

TypeScript uses **structural typing** (also called "duck typing at the type level"). If two types have the same shape, they're compatible -- even if they have different names.

```typescript
interface Dog {
  name: string;
  breed: string;
}

interface Pet {
  name: string;
  breed: string;
}

const dog: Dog = { name: "Rex", breed: "Lab" };
const pet: Pet = dog; // Works -- same shape
```

Python's type system is **nominal** by default -- a class `Dog` is not interchangeable with a class `Pet` even if they have identical fields. Python's `Protocol` class (from `typing`) gives you structural typing, but it's opt-in. In TypeScript, structural typing is the default and only mode.

---

## Basic Type Annotations

### Variables

```typescript
// TypeScript
let name: string = "Alice";
let age: number = 30;
let active: boolean = true;
let scores: number[] = [98, 85, 92];
```

```python
# Python equivalent
name: str = "Alice"
age: int = 30
active: bool = True
scores: list[int] = [98, 85, 92]
```

The syntax is similar: `name: Type` in both languages. The primitive type names differ:

| Python | TypeScript | Notes |
|---|---|---|
| `str` | `string` | Lowercase in TS |
| `int`, `float` | `number` | JS has a single number type |
| `bool` | `boolean` | Lowercase in TS |
| `None` | `null`, `undefined` | TS distinguishes these two (more in Module 01) |
| `list[int]` | `number[]` or `Array<number>` | Two equivalent syntaxes in TS |
| `dict[str, int]` | `Record<string, number>` | Or use an index signature (see below) |
| `tuple[int, str]` | `[number, string]` | TS tuples use bracket syntax |

**Index signatures** let you type objects with arbitrary keys — like Python's `dict[str, int]`:

```typescript
// Record shorthand
const scores: Record<string, number> = { alice: 95, bob: 87 };

// Index signature (equivalent, more verbose)
interface Scores {
  [key: string]: number;
}

// Useful when you have some known keys plus arbitrary ones:
interface HyperparameterConfig {
  learningRate: number;
  batchSize: number;
  [key: string]: number;  // allow any additional numeric params
}
```

### Functions

```typescript
// TypeScript
function greet(name: string, times: number): string {
  return `Hello, ${name}!`.repeat(times);
}

// Arrow function
const add = (a: number, b: number): number => a + b;

// void means the function doesn't return a value (Python's -> None)
function logMessage(msg: string): void {
  console.log(msg);
}
```

```python
# Python equivalent
def greet(name: str, times: int) -> str:
    return f"Hello, {name}!" * times

# Lambda (limited in Python)
add = lambda a, b: a + b  # Can't annotate lambdas in Python
```

### Type Inference

TypeScript infers types more aggressively than mypy. You often don't need to annotate variables:

```typescript
let name = "Alice";       // Inferred as string
let scores = [1, 2, 3];   // Inferred as number[]
const PI = 3.14;           // Inferred as the literal type 3.14 (not just number)

function double(x: number) {
  return x * 2;            // Return type inferred as number
}
```

**Rule of thumb:** annotate function parameters (they can't be inferred), but let TypeScript infer the rest. Add explicit return types on public API boundaries or when the inferred type isn't what you want.

When might the inferred type not be what you want?
- **You want a narrower type.** A function returns `"success"` or `"error"` but TS infers `string` instead of the union `"success" | "error"`.
- **You want to enforce a contract.** If the function body changes later and accidentally returns a different type, an explicit return type catches it at the function, not somewhere downstream.
- **The inferred type is unreadable.** Async functions can infer things like `Promise<{ name: string; score: number } | undefined>` — an explicit named type is clearer.

---

## Interfaces and Type Aliases

TypeScript has two ways to name an object shape: `interface` and `type`.

### Interfaces

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

function sendEmail(user: User): void {
  console.log(`Sending to ${user.email}`);
}
```

### Type Aliases

```typescript
type User = {
  id: number;
  name: string;
  email: string;
};
```

### When to Use Which

- Use `interface` for object shapes that might be extended. Interfaces can be extended and merged.
- Use `type` for unions, intersections, tuples, mapped types, or any type that isn't a plain object shape.

In practice, many teams just pick one and use it consistently. Both work for object shapes.

### Extending Interfaces

```typescript
interface Animal {
  name: string;
  species: string;
}

interface Pet extends Animal {
  owner: string;
  vaccinated: boolean;
}
```

This is like Python class inheritance for type purposes:

```python
# Python equivalent using TypedDict
class Animal(TypedDict):
    name: str
    species: str

class Pet(Animal):
    owner: str
    vaccinated: bool
```

### Optional and Readonly Properties

```typescript
interface Config {
  host: string;
  port: number;
  debug?: boolean;          // Optional -- may be undefined
  readonly version: string; // Cannot be reassigned after creation
}

const config: Config = { host: "localhost", port: 3000, version: "1.0" };
// config.version = "2.0"; // Error: Cannot assign to 'version'
```

Python equivalents:
- Optional: `debug: bool | None = None` in a dataclass, or `NotRequired` in TypedDict
- Readonly: no direct equivalent; Python uses conventions or `@property`

### Comparison to Python Types

| TypeScript | Python |
|---|---|
| `interface` | `TypedDict` (for dicts) or `Protocol` (for structural typing) |
| `type` alias | `TypeAlias` (3.10+) or `type` keyword (3.12+) |
| Optional `?` property | `NotRequired` in TypedDict, or `Optional` field |
| `extends` | Class inheritance on TypedDict/Protocol |
| `readonly` | No enforced equivalent |

---

## Union Types and Literal Types

### Union Types

```typescript
// TypeScript
function format(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}
```

```python
# Python equivalent
def format(value: str | int) -> str:  # Python 3.10+
    if isinstance(value, str):
        return value.upper()
    return f"{value:.2f}"
```

The syntax is nearly identical: `A | B` in both languages. In older Python, you'd write `Union[str, int]`.

### Literal Types

```typescript
type Status = "pending" | "running" | "completed" | "failed";
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

function setStatus(status: Status): void {
  // status can only be one of the four literal strings
}

setStatus("pending");  // OK
setStatus("unknown");  // Error: Argument of type '"unknown"' is not assignable
```

Python equivalent:

```python
from typing import Literal

Status = Literal["pending", "running", "completed", "failed"]

def set_status(status: Status) -> None: ...
```

### Discriminated Unions

This is a powerful TypeScript pattern with no clean Python equivalent. You define a union of object types, each with a shared property (the "discriminant") that tells them apart:

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number }
  | { kind: "triangle"; base: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height; // TS knows width/height exist here
    case "triangle":
      return 0.5 * shape.base * shape.height;
  }
}
```

TypeScript narrows the type inside each `case` branch. It knows that in the `"circle"` branch, `shape` has a `radius` property. This pattern is used extensively in real TypeScript code -- for API responses, state machines, Redux actions, and more.

The closest Python pattern would be a tagged union using dataclasses:

```python
@dataclass
class Circle:
    kind: Literal["circle"] = "circle"
    radius: float = 0

@dataclass
class Rectangle:
    kind: Literal["rectangle"] = "rectangle"
    width: float = 0
    height: float = 0

Shape = Circle | Rectangle
# Then use isinstance() or match/case to narrow
```

It works, but it's more verbose and the tooling support isn't as strong.

---

## Generics

Generics let you write functions and types that work with any type while preserving type information.

### Generic Functions

```typescript
// TypeScript
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const n = first([1, 2, 3]);      // n: number | undefined
const s = first(["a", "b"]);     // s: string | undefined
```

```python
# Python equivalent
from typing import TypeVar
T = TypeVar("T")

def first(arr: list[T]) -> T | None:
    return arr[0] if arr else None

# Or in Python 3.12+:
def first[T](arr: list[T]) -> T | None:
    return arr[0] if arr else None
```

The TS syntax `<T>` replaces Python's `TypeVar` boilerplate. Python 3.12 closed this gap with `def f[T](...)`, but most Python codebases still use the older `TypeVar` style.

### Generic Interfaces

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// Usage
type UserResponse = ApiResponse<User>;
type MetricsResponse = ApiResponse<Metric[]>;
```

```python
# Python equivalent
from typing import Generic, TypeVar
T = TypeVar("T")

class ApiResponse(Generic[T]):
    data: T
    status: int
    message: str
```

### Constraints

You can restrict what types a generic accepts:

```typescript
interface HasLength {
  length: number;
}

function longest<T extends HasLength>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}

longest("hello", "world");     // OK: strings have .length
longest([1, 2], [1, 2, 3]);   // OK: arrays have .length
longest(10, 20);               // Error: numbers don't have .length
```

Python equivalent using `Protocol`:

```python
class HasLength(Protocol):
    def __len__(self) -> int: ...

def longest(a: T, b: T) -> T:  # where T is bound to HasLength
    return a if len(a) >= len(b) else b
```

### Multiple Type Parameters

```typescript
function zip<A, B>(as: A[], bs: B[]): [A, B][] {
  return as.map((a, i) => [a, bs[i]]);
}

const pairs = zip([1, 2, 3], ["a", "b", "c"]); // [number, string][]
```

---

## Utility Types

TypeScript provides built-in utility types that transform existing types. These have no direct Python equivalents and are a major productivity feature.

### Partial\<T\>

Makes all properties optional:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// All fields become optional
type UserUpdate = Partial<User>;
// Equivalent to: { id?: number; name?: string; email?: string }

function updateUser(id: number, changes: Partial<User>): void {
  // Can pass any subset of User fields
}

updateUser(1, { name: "Alice" }); // OK -- only updating name
```

### Required\<T\>

The opposite -- makes all properties required:

```typescript
type Config = {
  host?: string;
  port?: number;
  debug?: boolean;
};

type StrictConfig = Required<Config>;
// All fields are now required
```

### Pick\<T, K\> and Omit\<T, K\>

Select or exclude specific properties:

```typescript
type UserPreview = Pick<User, "id" | "name">;
// { id: number; name: string }

type UserWithoutEmail = Omit<User, "email">;
// { id: number; name: string }
```

### Record\<K, V\>

Creates an object type with specified keys and value types:

```typescript
type Metrics = Record<string, number>;
// Equivalent to { [key: string]: number }

const scores: Metrics = {
  accuracy: 0.95,
  precision: 0.92,
  recall: 0.88,
};

// With literal key union:
type RGBColor = Record<"r" | "g" | "b", number>;
```

### Practical Example

```typescript
interface Experiment {
  id: string;
  name: string;
  model: string;
  hyperparams: Record<string, number>;
  metrics: Record<string, number>;
  status: "pending" | "running" | "completed" | "failed";
  createdAt: Date;
}

// For creating a new experiment -- don't need id or createdAt yet
type NewExperiment = Omit<Experiment, "id" | "createdAt">;

// For updating -- all fields optional
type ExperimentUpdate = Partial<Omit<Experiment, "id">>;

// For display -- just the summary fields
type ExperimentSummary = Pick<Experiment, "id" | "name" | "status">;
```

---

## Enums and Const Assertions

### String Enums

```typescript
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}

function move(dir: Direction): void { ... }
move(Direction.Up);
```

```python
# Python equivalent
from enum import Enum

class Direction(str, Enum):
    UP = "UP"
    DOWN = "DOWN"
    LEFT = "LEFT"
    RIGHT = "RIGHT"
```

### Numeric Enums

```typescript
enum LogLevel {
  Debug,    // 0
  Info,     // 1
  Warning,  // 2
  Error,    // 3
}
```

Numeric enums auto-increment from 0. This mirrors Python's `auto()`:

```python
class LogLevel(Enum):
    DEBUG = auto()    # 1 (Python auto() starts at 1)
    INFO = auto()
    WARNING = auto()
    ERROR = auto()
```

### Const Assertions (Preferred Alternative)

Many TypeScript developers prefer `as const` over enums:

```typescript
const DIRECTIONS = {
  Up: "UP",
  Down: "DOWN",
  Left: "LEFT",
  Right: "RIGHT",
} as const;

// Type: "UP" | "DOWN" | "LEFT" | "RIGHT"
type Direction = (typeof DIRECTIONS)[keyof typeof DIRECTIONS];
```

`as const` makes the object deeply readonly and narrows string values to literal types. It avoids some quirks of enums (like numeric enums being assignable to `number`) and produces simpler JavaScript output.

---

## Type Narrowing

Type narrowing is how TypeScript figures out a more specific type within a code block. You already do this in Python with `isinstance()` -- TypeScript has several mechanisms.

### typeof

```typescript
function process(value: string | number) {
  if (typeof value === "string") {
    // TypeScript knows value is string here
    console.log(value.toUpperCase());
  } else {
    // TypeScript knows value is number here
    console.log(value.toFixed(2));
  }
}
```

### instanceof

```typescript
class ApiError extends Error {
  statusCode: number;
  constructor(message: string, statusCode: number) {
    super(message);
    this.statusCode = statusCode;
  }
}

function handleError(err: Error) {
  if (err instanceof ApiError) {
    // TypeScript knows err is ApiError here
    console.log(`HTTP ${err.statusCode}: ${err.message}`);
  } else {
    console.log(`Error: ${err.message}`);
  }
}
```

### The `in` Operator

```typescript
interface Dog { bark(): void; }
interface Cat { meow(): void; }

function speak(animal: Dog | Cat) {
  if ("bark" in animal) {
    animal.bark(); // TypeScript knows it's Dog
  } else {
    animal.meow(); // TypeScript knows it's Cat
  }
}
```

### Discriminated Unions with switch

```typescript
type Result =
  | { status: "success"; data: string }
  | { status: "error"; message: string }
  | { status: "loading" };

function handleResult(result: Result) {
  switch (result.status) {
    case "success":
      console.log(result.data);    // TS knows data exists
      break;
    case "error":
      console.log(result.message); // TS knows message exists
      break;
    case "loading":
      console.log("Loading...");
      break;
  }
}
```

### Python Comparison

```python
# Python narrowing with isinstance
def process(value: str | int) -> None:
    if isinstance(value, str):
        print(value.upper())    # mypy knows it's str
    else:
        print(f"{value:.2f}")   # mypy knows it's int

# Python 3.10+ match/case
match result:
    case {"status": "success", "data": data}:
        print(data)
    case {"status": "error", "message": msg}:
        print(msg)
```

The concepts are the same; TypeScript just has more built-in narrowing mechanisms, and the compiler uses them more aggressively.

---

## Escape Hatches

### `any`

Opts out of type checking entirely. Equivalent to having no type hints at all in Python:

```typescript
let x: any = 5;
x = "hello";     // No error
x.foo.bar.baz;   // No error (but will crash at runtime)
```

**Avoid `any`** in production code. It defeats the purpose of TypeScript.

### `unknown`

The safe alternative to `any`. You can assign anything to `unknown`, but you must narrow it before using it:

```typescript
function process(input: unknown) {
  // input.toUpperCase();  // Error: Object is of type 'unknown'

  if (typeof input === "string") {
    input.toUpperCase(); // OK -- narrowed to string
  }
}
```

Think of `unknown` as "I don't know what this is yet, but I'll check before using it." Use `unknown` for values from external sources (API responses, user input, JSON parsing).

### `never`

Represents values that should never occur. Useful for exhaustiveness checking:

```typescript
type Shape = "circle" | "square" | "triangle";

function area(shape: Shape): number {
  switch (shape) {
    case "circle":
      return 0; // placeholder
    case "square":
      return 0;
    case "triangle":
      return 0;
    default:
      // If you add a new Shape variant but forget to handle it,
      // TypeScript will error here:
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

### Type Assertions

When you know more than the compiler:

```typescript
const input = document.getElementById("search") as HTMLInputElement;
input.value = "hello"; // TS now knows this is an input element

// Alternative syntax (not usable in JSX/TSX files):
const input2 = <HTMLInputElement>document.getElementById("search");
```

Use assertions sparingly. They're a way of telling the compiler "trust me" -- and you might be wrong.

---

## Compiler Configuration

TypeScript is configured via `tsconfig.json` in your project root. We'll cover this fully in Module 12; here's what you need to know now.

### Minimal tsconfig.json

```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```

### Key Options

- **`strict: true`** -- Enables all strict type checking options. Always use this. It's like running mypy in strict mode.
- **`target`** -- The JavaScript version to compile to. `ES2022` is a good modern default.
- **`module`** -- The module system for output. `NodeNext` for Node.js projects.

### Running TypeScript

```bash
# Type-check without emitting JS (like running mypy)
npx tsc --noEmit

# Compile TS to JS
npx tsc

# Run a TS file directly (uses tsx, a TS runner)
npx tsx src/main.ts
```

`npx tsc` is the type checker. `npx tsx` is a convenience tool that type-checks and runs in one step (similar to how you might use `python` directly rather than running mypy separately).

---

## Further Resources

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/) -- Official documentation, thorough and well-written
- [TypeScript Playground](https://www.typescriptlang.org/play) -- Try TypeScript in the browser with instant feedback
- [Type Challenges](https://github.com/type-challenges/type-challenges) -- Practice problems for the type system (come back to this after finishing the exercises below)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/) -- Free online book with detailed explanations

---

## Exercises

1. [Type Annotations](exercises/01-type-annotations.md) -- Add types to untyped JavaScript functions
2. [Interfaces and Types](exercises/02-interfaces-and-types.md) -- Define types for an ML experiment tracker
3. [Generics](exercises/03-generics.md) -- Implement generic utility types and functions
4. [Type a Python Script](exercises/04-type-a-python-script.md) -- Rewrite a typed Python script in TypeScript
