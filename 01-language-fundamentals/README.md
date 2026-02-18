# Module 01: Language Fundamentals

Core JavaScript syntax and semantics, mapped from Python equivalents. This module covers everything you need to read and write everyday JS.

**Prerequisites:** [Module 00](../00-js-in-5-minutes/README.md)
**Time:** 2-3 sessions

## Variables and Declarations

Python has one way to assign variables. JavaScript has three keywords — but you only need two:

```python
# Python
name = "Alice"      # Rebindable
name = "Bob"        # Fine
```

```javascript
// JavaScript
const name = "Alice";  // Cannot be reassigned — use this by default
let name = "Bob";      // Can be reassigned — use when you need to
var name = "Charlie";  // Old style — never use this (function-scoped, hoisted, leaks)
```

**Rule:** Use `const` by default. Use `let` when you need to reassign. Never use `var`.

`const` doesn't mean immutable — it means the *binding* can't change. The value itself can still be mutated:

```javascript
const items = [1, 2, 3];
items.push(4);          // Fine — mutating the array
items = [5, 6];         // Error — can't reassign the variable
```

This is different from Python, where there's no concept of a constant binding at all.

## Primitive Types

JavaScript has 7 primitive types. Python has fewer distinct ones:

| JavaScript | Python equivalent | Notes |
|-----------|------------------|-------|
| `string` | `str` | Same concept |
| `number` | `int` + `float` | JS has ONE number type (64-bit float) |
| `boolean` | `bool` | `true`/`false` (lowercase!) |
| `null` | `None` | Explicit "no value" |
| `undefined` | *(no equivalent)* | Variable declared but not assigned |
| `bigint` | `int` (arbitrary precision) | For large integers: `123n` |
| `symbol` | *(no equivalent)* | Unique identifiers, rarely used directly |

The `undefined` vs `null` distinction trips up everyone. In practice:
- `undefined` = "hasn't been set yet" (missing property, uninitialized variable)
- `null` = "explicitly set to nothing"
- You'll encounter `undefined` far more often

### typeof

```javascript
typeof "hello"      // "string"
typeof 42           // "number"
typeof true         // "boolean"
typeof undefined    // "undefined"
typeof null         // "object"    ← 30-year-old bug, can never be fixed
typeof []           // "object"    ← arrays are objects
typeof {}           // "object"

Array.isArray([])   // true — use this to check for arrays
```

## Truthiness

This is where JS and Python diverge in surprising ways:

```python
# Python falsy: None, False, 0, 0.0, "", [], {}, set()
bool([])    # False
bool({})    # False
```

```javascript
// JS falsy: false, 0, -0, 0n, "", null, undefined, NaN
// THAT'S IT. Everything else is truthy.
Boolean([])    // true!  ← empty arrays are truthy (Python: False)
Boolean({})    // true!  ← empty objects are truthy (Python: False)

// So this is WRONG:
if (myArray) { /* runs even if array is empty! */ }

// Do this instead:
if (myArray.length) { /* runs only if array has items */ }
```

### Equality: Always Use `===`

```javascript
// == does type coercion (unpredictable)
0 == ""         // true   ← WAT
0 == false      // true
"" == false     // true
null == undefined // true ← the one useful case

// === does strict comparison (predictable)
0 === ""        // false
0 === false     // false

// Rule: always use === and !==
```

## Strings

Template literals (backticks) are JavaScript's answer to Python f-strings:

```python
# Python
name = "world"
greeting = f"Hello, {name}!"
multiline = """
  line 1
  line 2
"""
```

```javascript
// JavaScript
const name = "world";
const greeting = `Hello, ${name}!`;   // backticks + ${} (not f"")
const multiline = `
  line 1
  line 2
`;

// Regular strings use single or double quotes (no difference)
const a = "hello";
const b = 'hello';  // Same thing — pick one style and stick with it
```

Common string methods compared:

| Python | JavaScript | Notes |
|--------|-----------|-------|
| `s.upper()` | `s.toUpperCase()` | |
| `s.lower()` | `s.toLowerCase()` | |
| `s.strip()` | `s.trim()` | |
| `s.startswith("x")` | `s.startsWith("x")` | camelCase |
| `s.endswith("x")` | `s.endsWith("x")` | |
| `s.replace("a", "b")` | `s.replace("a", "b")` | Only first! Use `replaceAll` for all |
| `s.split(",")` | `s.split(",")` | Same |
| `",".join(list)` | `arr.join(",")` | Method is on the array, not the separator |
| `"x" in s` | `s.includes("x")` | |
| `len(s)` | `s.length` | Property, not function |

## Arrays

Arrays are JavaScript's lists. Most operations have close Python equivalents, but the syntax differs:

### Core Methods

```javascript
const nums = [1, 2, 3, 4, 5];

// Adding/removing
nums.push(6);          // Append (like list.append())
nums.pop();            // Remove last (like list.pop())
nums.unshift(0);       // Prepend (like list.insert(0, x))
nums.shift();          // Remove first

// Checking
nums.includes(3);      // true (like `3 in nums` in Python)
nums.indexOf(3);       // 2 (like nums.index(3))
nums.length;           // Property, not len()
```

### Functional Methods (replacing list comprehensions)

Python uses list comprehensions. JavaScript uses method chains:

```python
# Python
nums = [1, 2, 3, 4, 5]
doubled = [x * 2 for x in nums]              # [2, 4, 6, 8, 10]
evens = [x for x in nums if x % 2 == 0]      # [2, 4]
total = sum(nums)                              # 15
first_even = next(x for x in nums if x % 2 == 0)  # 2
```

```javascript
// JavaScript
const nums = [1, 2, 3, 4, 5];
const doubled = nums.map(x => x * 2);              // [2, 4, 6, 8, 10]
const evens = nums.filter(x => x % 2 === 0);       // [2, 4]
const total = nums.reduce((sum, x) => sum + x, 0); // 15
const firstEven = nums.find(x => x % 2 === 0);     // 2
```

Full comparison:

| Python | JavaScript | Returns |
|--------|-----------|---------|
| `[f(x) for x in lst]` | `lst.map(x => f(x))` | New array |
| `[x for x in lst if p(x)]` | `lst.filter(x => p(x))` | New array |
| `sum(lst)` / `functools.reduce` | `lst.reduce((acc, x) => ..., init)` | Single value |
| `next(x for x if p(x))` | `lst.find(x => p(x))` | First match or undefined |
| `any(p(x) for x in lst)` | `lst.some(x => p(x))` | Boolean |
| `all(p(x) for x in lst)` | `lst.every(x => p(x))` | Boolean |
| `for x in lst:` | `lst.forEach(x => ...)` | Nothing (side effects) |

### Slicing and Destructuring

```javascript
// No negative indexing with []
nums[nums.length - 1];  // Last element
nums.at(-1);             // Same, using .at() (modern)

// Slicing
nums.slice(1, 3);        // [2, 3] — like nums[1:3]
nums.slice(1);           // [2, 3, 4, 5] — like nums[1:]

// Destructuring (no Python equivalent, very common in JS)
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first = 1, second = 2, rest = [3, 4, 5]

// Spread operator (like Python's *unpacking)
const combined = [...arr1, ...arr2];   // Like [*list1, *list2]
const copy = [...nums];               // Shallow copy, like list(nums)
```

### Sorting

```javascript
// WARNING: .sort() mutates the original array AND sorts lexicographically
const nums = [10, 2, 30, 1];
nums.sort();                        // [1, 10, 2, 30] — alphabetical!
nums.sort((a, b) => a - b);        // [1, 2, 10, 30] — numeric

// Non-mutating sort (ES2023+):
const sorted = nums.toSorted((a, b) => a - b);
// Or: const sorted = [...nums].sort((a, b) => a - b);
```

## Objects

JavaScript objects are like Python dicts, but with some key differences:

```python
# Python dict
person = {"name": "Alice", "age": 30}
person["name"]                          # "Alice"
person.get("email", "N/A")             # "N/A"
"name" in person                        # True
```

```javascript
// JavaScript object
const person = { name: "Alice", age: 30 };  // Keys don't need quotes
person.name;                                 // "Alice" — dot notation
person["name"];                              // "Alice" — bracket notation (same)
person.email ?? "N/A";                       // "N/A" — nullish coalescing
"name" in person;                            // true
```

### Object Utilities

```javascript
Object.keys(person);      // ["name", "age"]
Object.values(person);    // ["Alice", 30]
Object.entries(person);   // [["name", "Alice"], ["age", 30]]
```

### Spread and Destructuring

```javascript
// Spread (like Python's ** unpacking)
const updated = { ...person, email: "a@b.com" };  // Merge/extend

// Destructuring (extract named fields)
const { name, age } = person;  // name = "Alice", age = 30

// Rename while destructuring
const { name: firstName } = person;  // firstName = "Alice"

// Default values
const { email = "unknown" } = person;  // email = "unknown"
```

### Optional Chaining and Nullish Coalescing

These have no direct Python equivalent and are used constantly in JS:

```javascript
// Optional chaining — safe navigation for null/undefined
const city = user?.address?.city;          // undefined if any part is null/undefined
const first = items?.[0];                  // Safe array access
const result = obj?.method?.();            // Safe method call

// Nullish coalescing — default only for null/undefined
const name = input ?? "default";           // "default" only if input is null/undefined
// vs
const name2 = input || "default";          // "default" for ANY falsy value (0, "", false)
```

## Functions

JavaScript has three ways to write functions. Python has two (`def` and `lambda`):

```javascript
// 1. Function declaration (like Python's def)
function greet(name) {
  return `Hello, ${name}!`;
}

// 2. Function expression (assigning to a variable)
const greet = function(name) {
  return `Hello, ${name}!`;
};

// 3. Arrow function (like Python's lambda, but can have bodies)
const greet = (name) => `Hello, ${name}!`;         // Implicit return
const greet = (name) => {                           // With body
  return `Hello, ${name}!`;
};
```

**When to use each:**
- Arrow functions for callbacks, short functions, and most new code
- Function declarations for top-level functions you want hoisted
- You'll rarely use function expressions

### Default Parameters and Rest

```javascript
// Default parameters (same as Python)
function greet(name = "world") {
  return `Hello, ${name}!`;
}

// Rest parameters (like Python's *args)
function sum(...numbers) {          // numbers is an array
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3);  // 6
```

### Closures

Closures work similarly to Python — inner functions capture variables from the enclosing scope:

```javascript
function makeCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    getCount: () => count,
  };
}

const counter = makeCounter();
counter.increment();    // 1
counter.increment();    // 2
counter.getCount();     // 2
```

The key difference from Python: JS closures can freely modify captured variables with `let`. In Python, you need the `nonlocal` keyword to reassign a variable from an enclosing scope — without it, assignment creates a new local variable instead.

## Classes

ES6 classes are syntactically similar to Python classes:

```python
# Python
class Experiment:
    def __init__(self, name: str, model: str):
        self.name = name
        self.model = model
        self.runs: list[dict] = []

    def add_run(self, metrics: dict) -> None:
        self.runs.append(metrics)

    def best_accuracy(self) -> float:
        return max(r["accuracy"] for r in self.runs)

class DeepLearningExperiment(Experiment):
    def __init__(self, name: str, model: str, framework: str):
        super().__init__(name, model)
        self.framework = framework
```

```javascript
// JavaScript
class Experiment {
  constructor(name, model) {
    this.name = name;
    this.model = model;
    this.runs = [];
  }

  addRun(metrics) {
    this.runs.push(metrics);
  }

  bestAccuracy() {
    return Math.max(...this.runs.map(r => r.accuracy));
  }
}

class DeepLearningExperiment extends Experiment {
  constructor(name, model, framework) {
    super(name, model);      // Must call super() before using this
    this.framework = framework;
  }
}
```

Key differences:
- `constructor` instead of `__init__`
- `this` instead of `self` (and `this` is implicit — not in the parameter list)
- `extends` instead of `class Child(Parent)`
- No `def` keyword — just `methodName() { }`
- camelCase method names (JS convention) vs snake_case (Python convention)

### Private Fields

```javascript
class User {
  #password;          // Private field (truly private, not just convention)

  constructor(name, password) {
    this.name = name;       // Public
    this.#password = password;
  }

  checkPassword(input) {
    return input === this.#password;
  }
}
```

Python uses `_` convention for private; JavaScript `#` fields are enforced by the runtime.

## Error Handling

Very similar to Python, with slightly different syntax:

```python
# Python
try:
    result = parse_config(data)
except ValueError as e:
    print(f"Invalid: {e}")
except Exception as e:
    print(f"Error: {e}")
    raise
finally:
    cleanup()

raise ValueError("bad input")
```

```javascript
// JavaScript
try {
  const result = parseConfig(data);
} catch (error) {
  if (error instanceof TypeError) {
    console.log(`Invalid: ${error.message}`);
  } else {
    console.log(`Error: ${error}`);
    throw error;    // Re-throw (like Python's raise)
  }
} finally {
  cleanup();
}

throw new Error("bad input");   // throw + new, not raise
```

Key differences:
- `catch` takes one parameter (not typed — you check with `instanceof`)
- You `throw new Error(...)`, not `raise ValueError(...)`
- Standard error types: `Error`, `TypeError`, `RangeError`, `SyntaxError`
- You can throw anything (not just Error objects), but you shouldn't

## Control Flow

### if/else

Same logic, different syntax (braces instead of colons + indentation):

```javascript
if (score > 90) {
  console.log("A");
} else if (score > 80) {    // else if, not elif
  console.log("B");
} else {
  console.log("C");
}
```

### Loops

```javascript
// Iterating over values (like Python's for x in list)
for (const item of items) {
  console.log(item);
}

// WARNING: for...in iterates over KEYS (like Python dict iteration)
for (const key in obj) {
  console.log(key, obj[key]);
}
// Don't use for...in on arrays — it gives you string indices

// C-style for loop (no Python equivalent)
for (let i = 0; i < 10; i++) {
  console.log(i);
}

// While loop (same concept)
while (condition) {
  doSomething();
}
```

### switch (Python 3.10+ match/case equivalent)

```javascript
switch (status) {
  case "pending":
    handlePending();
    break;           // Must break! Otherwise falls through to next case
  case "running":
    handleRunning();
    break;
  default:
    handleUnknown();
}
```

### Ternary Operator

```python
# Python
result = "yes" if condition else "no"
```

```javascript
// JavaScript
const result = condition ? "yes" : "no";
```

## Iterators and Generators

Brief coverage — you'll use these occasionally, not daily:

```javascript
// Generator function (like Python's yield)
function* range(start, end) {
  for (let i = start; i < end; i++) {
    yield i;
  }
}

for (const n of range(0, 5)) {
  console.log(n);  // 0, 1, 2, 3, 4
}

// Spread a generator into an array
const nums = [...range(0, 5)];  // [0, 1, 2, 3, 4]
```

Generators are used less in JS than Python because JS favors array methods (map/filter/reduce) over lazy iteration. You'll encounter them most when working with async iterators (Module 03).

## Further Reading

- [javascript.info — JavaScript Fundamentals](https://javascript.info/first-steps) — Detailed coverage of everything in this module
- [MDN JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) — Reference documentation
- [You Don't Know JS Yet](https://github.com/getify/You-Dont-Know-JS) — Deep dive into "why" things work the way they do

## Exercises

1. [Types and Variables](./exercises/01-types-and-variables.md) — Predict outputs, verify in Node
2. [Functions and Closures](./exercises/02-functions-and-closures.md) — Rewrite a Python data pipeline
3. [Objects and Classes](./exercises/03-objects-and-classes.md) — Build an experiment tracker
4. [Error Handling](./exercises/04-error-handling.md) — Write a config parser

Next: [Module 02 — TypeScript Essentials](../02-typescript-essentials/README.md)
