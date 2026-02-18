# Common Gotchas for Python Developers

Things that will trip you up coming from Python. Bookmark this and revisit when something feels wrong.

## 1. `==` vs `===` (Always Use `===`)

```typescript
// JavaScript has two equality operators
0 == ""        // true  (type coercion — WAT)
0 === ""       // false (strict equality — what you expect)
null == undefined  // true  (this is actually the ONE useful case of ==)
null === undefined // false

// Rule: always use === and !==. The only exception is `x == null`
// which catches both null and undefined (a common pattern).
```

Python's `==` compares values sensibly. JavaScript's `==` does wild type coercion. Always use `===`.

## 2. `undefined` vs `null`

Python has one "nothing" value: `None`. JavaScript has two:

```typescript
let x;              // x is undefined (declared but not assigned)
let y = null;       // y is null (explicitly set to "no value")
let z = undefined;  // Also valid but unusual

// Both are falsy. Check for either with:
if (x == null) { }  // Catches both null and undefined
```

You'll mostly encounter `undefined` from missing object properties and uninitialized variables. Use `null` when you want to explicitly say "no value."

## 3. Truthiness is Different

```python
# Python falsy values: None, False, 0, 0.0, "", [], {}, set()
bool([])    # False
bool({})    # False
```

```typescript
// JavaScript falsy values: false, 0, -0, 0n, "", null, undefined, NaN
// THAT'S IT. Empty arrays and objects are TRUTHY.
Boolean([])    // true!  (Python: False)
Boolean({})    // true!  (Python: False)

// This means:
if (myArray) { }       // True even for empty array!
if (myArray.length) { } // Check length instead
```

## 4. No Negative Indexing

```python
# Python
items[-1]    # Last element
items[-2]    # Second to last
items[1:3]   # Slice
```

```typescript
// TypeScript — none of these work with []
items[items.length - 1];  // Last element
items.at(-1);              // ES2022: .at() supports negative indexes!
items.slice(1, 3);         // Slice (method, not syntax)
```

## 5. `this` is Not `self`

Python's `self` is explicit and predictable. JavaScript's `this` depends on *how a function is called*, not where it's defined.

```typescript
class Timer {
  seconds = 0;

  start() {
    // BUG: `this` inside setInterval callback is not the Timer instance
    setInterval(function() {
      this.seconds++;  // `this` is undefined or the global object!
    }, 1000);

    // FIX: Arrow functions inherit `this` from the enclosing scope
    setInterval(() => {
      this.seconds++;  // Works! Arrow function keeps Timer's `this`
    }, 1000);
  }
}
```

**Rule of thumb:** Use arrow functions for callbacks. Use regular functions for methods and constructors.

## 6. `for...in` vs `for...of`

```typescript
const arr = ["a", "b", "c"];

// WRONG — for...in iterates over KEYS (indexes as strings!)
for (const item in arr) {
  console.log(item);  // "0", "1", "2" — not the values!
}

// RIGHT — for...of iterates over VALUES (like Python's for...in)
for (const item of arr) {
  console.log(item);  // "a", "b", "c"
}
```

Python's `for x in list` is JavaScript's `for (const x of list)`. JavaScript's `for...in` is for object keys (and even then, `Object.keys()` is usually better).

## 7. `.sort()` Mutates (and Sorts Lexicographically)

```typescript
const nums = [10, 2, 30, 1];

// Python: sorted(nums) returns new list, nums.sort() mutates in place
// JS: .sort() ALWAYS mutates the original array

nums.sort();              // [1, 10, 2, 30] — lexicographic! WAT.
nums.sort((a, b) => a - b);  // [1, 2, 10, 30] — numeric sort

// To sort without mutating:
const sorted = [...nums].sort((a, b) => a - b);
// Or in ES2023+:
const sorted2 = nums.toSorted((a, b) => a - b);  // Returns new array
```

## 8. Objects Are Not Dicts

Python dicts and JS objects look similar but behave differently:

```typescript
// Keys are always strings (or Symbols)
const obj = { 1: "one" };
obj[1];     // "one"
obj["1"];   // "one" — same thing! Key was coerced to string

// No .keys(), .values(), .items() methods on the object itself
Object.keys(obj);     // ["1"]
Object.values(obj);   // ["one"]
Object.entries(obj);  // [["1", "one"]]

// For actual dict-like behavior with any key type, use Map:
const map = new Map<number, string>();
map.set(1, "one");
map.get(1);           // "one"
map.size;             // 1
```

## 9. Semicolons Are Optional (But Sometimes Bite)

JavaScript has Automatic Semicolon Insertion (ASI). You can write code without semicolons and it usually works. But:

```typescript
// This breaks without a semicolon on the previous line:
const x = 42
[1, 2, 3].forEach(n => console.log(n))
// JS reads this as: const x = 42[1, 2, 3].forEach(...)

// This also breaks:
const fn = () => "hello"
(async () => { })()
// JS reads this as: const fn = () => "hello"(async () => { })()
```

**Recommendation:** Use a formatter like Prettier and let it handle semicolons for you. If writing without a formatter, add semicolons consistently.

## 10. `typeof null === "object"`

```typescript
typeof "hello"     // "string"
typeof 42          // "number"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof null        // "object"  — a 30-year-old bug that can never be fixed
typeof []          // "object"  — arrays are objects
typeof {}          // "object"

// To check for arrays:
Array.isArray([])  // true
Array.isArray({})  // false
```

## 11. Floating Point (Same as Python, but More Common)

```typescript
0.1 + 0.2 === 0.3   // false (0.30000000000000004)
// Same in Python! But you encounter it more in JS because there's no int type.

// Every number is a 64-bit float. Large integers lose precision:
9007199254740992 === 9007199254740993   // true! Both are beyond safe range

// Use BigInt for large integers:
const big = 9007199254740993n;  // Note the 'n' suffix
```

## 12. Callbacks and Closures in Loops

```typescript
// Classic gotcha (with var — but you should never use var)
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Prints: 3, 3, 3 (not 0, 1, 2)

// Fixed with let (block-scoped)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Prints: 0, 1, 2
```

This is why you always use `let` or `const`, never `var`.

## 13. JSON.parse() Returns `any`

```typescript
const data = JSON.parse('{"name": "Alice"}');
// data is type `any` — TypeScript has no idea what's inside

// Always validate or assert:
const data = JSON.parse(input) as { name: string };  // Type assertion (trust me)

// Better: use a validation library like Zod
import { z } from "zod";
const schema = z.object({ name: z.string() });
const data = schema.parse(JSON.parse(input));  // Runtime validation + types
```

## 14. No Tuple Type at Runtime

```python
# Python
point = (1, 2)            # Immutable tuple
x, y = point              # Unpack
```

```typescript
// TypeScript has tuple TYPES but they're just arrays at runtime
const point: [number, number] = [1, 2];  // Type-checked as 2-element
const [x, y] = point;                     // Destructure (same as Python unpack)
point.push(3);                             // TypeScript won't stop you at runtime!
```

## 15. Async Errors Can Be Silently Swallowed

```typescript
// This error disappears silently:
async function doSomething() {
  throw new Error("oops");
}
doSomething();  // No await, no .catch() — error is just... gone

// Always await or catch:
await doSomething();                    // Error propagates
doSomething().catch(console.error);     // Error is logged
```

In Python, unhandled async errors at least produce a warning. Node.js will crash on unhandled rejections (which is actually better — fail loudly).
