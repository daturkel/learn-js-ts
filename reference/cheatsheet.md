# JS/TS vs Python Cheatsheet

Keep this open while working through modules. Shows the most common patterns side by side.

## Variables

```python
# Python
name = "Alice"          # Reassignable
PI = 3.14               # Convention only, still mutable
```

```typescript
// TypeScript
let name = "Alice";       // Reassignable
const PI = 3.14;          // Cannot be reassigned (use by default)
// var name = "Alice";    // Old style — never use this
```

## Strings

```python
# Python
name = "world"
greeting = f"Hello, {name}!"
multiline = """
  This spans
  multiple lines
"""
"hello".upper()
"hello world".split(" ")
"hello" in "hello world"     # True
len("hello")                 # 5
```

```typescript
// TypeScript
const name = "world";
const greeting = `Hello, ${name}!`;   // Backticks, not f""
const multiline = `
  This spans
  multiple lines
`;
"hello".toUpperCase();
"hello world".split(" ");
"hello world".includes("hello");       // true (no 'in' operator for strings)
"hello".length;                        // 5 (property, not function)
```

## Numbers

```python
# Python
x = 42          # int
y = 3.14        # float
z = 1_000_000   # underscore separators
10 // 3         # 3 (integer division)
10 % 3          # 1
2 ** 10         # 1024
```

```typescript
// TypeScript
const x = 42;          // number (no int/float distinction)
const y = 3.14;        // number
const z = 1_000_000;   // underscore separators (same!)
Math.floor(10 / 3);    // 3 (no // operator)
10 % 3;                // 1
2 ** 10;               // 1024 (same!)
```

## Booleans & Nulls

```python
# Python
True, False
None
x is None
```

```typescript
// TypeScript
true, false                  // lowercase!
null                         // explicit "no value"
undefined                    // variable declared but not assigned
x === null
x === undefined
x == null                    // true for both null and undefined (rare useful case of ==)
```

## Arrays (Lists)

```python
# Python
nums = [1, 2, 3]
nums.append(4)
nums[0]                     # 1
nums[-1]                    # 3
nums[1:3]                   # [2, 3]
len(nums)                   # 4
3 in nums                   # True
[x * 2 for x in nums]      # [2, 4, 6, 8]
[x for x in nums if x > 2] # [3, 4]
sorted(nums, reverse=True)  # [4, 3, 2, 1]
```

```typescript
// TypeScript
const nums = [1, 2, 3];
nums.push(4);
nums[0];                       // 1
nums[nums.length - 1];         // 3 (no negative indexing)
nums.slice(1, 3);              // [2, 3]
nums.length;                   // 4 (property, not function)
nums.includes(3);              // true
nums.map(x => x * 2);         // [2, 4, 6, 8]
nums.filter(x => x > 2);      // [3, 4]
[...nums].sort((a, b) => b - a); // [4, 3, 2, 1] (sort mutates! spread first)
```

## Objects (Dicts)

```python
# Python
person = {"name": "Alice", "age": 30}
person["name"]                    # "Alice"
person.get("email", "N/A")       # "N/A"
"name" in person                  # True
person.keys()
person.values()
person.items()
{**person, "email": "a@b.com"}   # Merge
```

```typescript
// TypeScript
const person = { name: "Alice", age: 30 };  // No quotes needed on keys
person.name;                     // "Alice" (dot notation preferred)
person["name"];                  // Also works
person.email ?? "N/A";           // Nullish coalescing (like .get() with default)
"name" in person;                // true
Object.keys(person);
Object.values(person);
Object.entries(person);
{ ...person, email: "a@b.com" }; // Spread to merge
```

## Functions

```python
# Python
def greet(name: str, excited: bool = False) -> str:
    if excited:
        return f"HI {name}!!!"
    return f"Hello, {name}"

# Lambda
double = lambda x: x * 2
```

```typescript
// TypeScript
function greet(name: string, excited: boolean = false): string {
  if (excited) {
    return `HI ${name}!!!`;
  }
  return `Hello, ${name}`;
}

// Arrow function (preferred for short functions / callbacks)
const double = (x: number): number => x * 2;

// Arrow with body
const greet2 = (name: string): string => {
  return `Hello, ${name}`;
};
```

## Destructuring (no direct Python equivalent)

```typescript
// TypeScript — destructuring is everywhere
const { name, age } = person;              // Extract from object
const [first, ...rest] = [1, 2, 3, 4];    // first=1, rest=[2,3,4]
const { name, ...others } = person;        // name="Alice", others={age: 30}

// In function parameters
function greet({ name, age }: { name: string; age: number }) {
  return `${name} is ${age}`;
}
```

```python
# Python — closest equivalents
name, age = person["name"], person["age"]  # Manual
first, *rest = [1, 2, 3, 4]               # Tuple unpacking (similar!)
```

## Control Flow

```python
# Python
if x > 0:
    print("positive")
elif x == 0:
    print("zero")
else:
    print("negative")

for item in items:
    print(item)

for i, item in enumerate(items):
    print(i, item)

while condition:
    do_something()
```

```typescript
// TypeScript
if (x > 0) {
  console.log("positive");
} else if (x === 0) {       // === not ==
  console.log("zero");
} else {
  console.log("negative");
}

for (const item of items) {  // for...of (not for...in!)
  console.log(item);
}

items.forEach((item, i) => {  // With index
  console.log(i, item);
});

while (condition) {
  doSomething();
}
```

## Classes

```python
# Python
class Dog:
    def __init__(self, name: str, breed: str):
        self.name = name
        self.breed = breed

    def bark(self) -> str:
        return f"{self.name} says woof!"

class Puppy(Dog):
    def bark(self) -> str:
        return f"{self.name} says yip!"
```

```typescript
// TypeScript
class Dog {
  name: string;
  breed: string;

  constructor(name: string, breed: string) {
    this.name = name;
    this.breed = breed;
  }

  bark(): string {
    return `${this.name} says woof!`;
  }
}

class Puppy extends Dog {
  bark(): string {
    return `${this.name} says yip!`;
  }
}
```

## Error Handling

```python
# Python
try:
    result = risky_operation()
except ValueError as e:
    print(f"Bad value: {e}")
except Exception as e:
    print(f"Something went wrong: {e}")
finally:
    cleanup()

# Raising
raise ValueError("invalid input")
```

```typescript
// TypeScript
try {
  const result = riskyOperation();
} catch (error) {
  if (error instanceof TypeError) {
    console.log(`Bad type: ${error.message}`);
  } else {
    console.log(`Something went wrong: ${error}`);
  }
} finally {
  cleanup();
}

// Throwing
throw new Error("invalid input");
```

## Async / Await

```python
# Python
import asyncio

async def fetch_data(url: str) -> dict:
    response = await httpx.get(url)
    return response.json()

async def main():
    results = await asyncio.gather(
        fetch_data("https://api1.com"),
        fetch_data("https://api2.com"),
    )

asyncio.run(main())
```

```typescript
// TypeScript
async function fetchData(url: string): Promise<Record<string, unknown>> {
  const response = await fetch(url);
  return response.json();
}

async function main() {
  const results = await Promise.all([
    fetchData("https://api1.com"),
    fetchData("https://api2.com"),
  ]);
}

main();   // No special runner needed — Node runs async natively
```

## Imports

```python
# Python
import os
from pathlib import Path
from mymodule import MyClass, helper_function
```

```typescript
// TypeScript
import os from "node:os";                          // Node built-in
import { readFile } from "node:fs/promises";       // Named import
import { MyClass, helperFunction } from "./mymodule"; // Local file (needs ./ prefix)
import chalk from "chalk";                         // npm package
```

## Type Annotations

```python
# Python (type hints)
name: str = "Alice"
nums: list[int] = [1, 2, 3]
person: dict[str, Any] = {"name": "Alice"}
callback: Callable[[int], str] = str

def process(items: list[str], verbose: bool = False) -> dict[str, int]:
    ...

class Config(TypedDict):
    name: str
    debug: bool
```

```typescript
// TypeScript
const name: string = "Alice";
const nums: number[] = [1, 2, 3];
const person: Record<string, unknown> = { name: "Alice" };
const callback: (n: number) => string = String;

function process(items: string[], verbose: boolean = false): Record<string, number> {
  // ...
}

interface Config {
  name: string;
  debug: boolean;
}
```

## Common Patterns

```python
# Python — ternary
result = "yes" if condition else "no"

# Python — unpacking / spread
merged = {**dict1, **dict2}
combined = [*list1, *list2]

# Python — optional chaining (3.10+ match, but usually)
value = obj.attr if obj else None
```

```typescript
// TypeScript — ternary
const result = condition ? "yes" : "no";

// TypeScript — spread
const merged = { ...obj1, ...obj2 };
const combined = [...arr1, ...arr2];

// TypeScript — optional chaining
const value = obj?.attr;              // undefined if obj is null/undefined
const value2 = obj?.method?.();       // Safe method call
const value3 = arr?.[0];             // Safe array access

// TypeScript — nullish coalescing
const name = input ?? "default";      // Only null/undefined trigger default
// vs
const name2 = input || "default";     // Any falsy value triggers default (0, "", false too)
```
