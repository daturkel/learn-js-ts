# Exercise 01: REPL Exploration

**Time:** ~20 minutes

Get familiar with JavaScript syntax by experimenting in the Node.js REPL, comparing what you already know from Python.

## Setup

Run `node` in your terminal to start the Node.js REPL. It works just like Python's — type expressions, get results. Press `Ctrl+D` or type `.exit` to quit.

## Exercises

Try each expression in the Node REPL. Before running it, predict what the output will be based on your Python knowledge. Pay attention to where JS diverges.

### 1. Basic output

```js
console.log("Hello from JavaScript");
```

Expected output: `Hello from JavaScript`

In Python this is `print()`. Note: `console.log` returns `undefined` (you'll see that printed too in the REPL).

### 2. Template literals (JS equivalent of f-strings)

```js
const name = "world";
`Hello, ${name}!`;
```

Expected output: `'Hello, world!'`

<details>
<summary>What's different from Python?</summary>

- Uses backticks `` ` `` instead of `f""`
- Uses `${}` instead of `{}`
- Backtick strings also support multiline without any special syntax
</details>

### 3. Variable declarations

```js
let x = 10;
x = 20;
x;
```

Expected output: `20`

Now try:

```js
const y = 10;
y = 20;
```

Expected output: `TypeError: Assignment to constant variable.`

<details>
<summary>Python comparison</summary>

Python has no equivalent of `const`. The convention of `UPPER_CASE` for constants is not enforced. In JS, `const` actually prevents reassignment. Use `const` by default; use `let` only when you need to reassign.
</details>

### 4. Arrow functions

```js
const double = (x) => x * 2;
double(5);
```

Expected output: `10`

```js
const add = (a, b) => a + b;
add(3, 4);
```

Expected output: `7`

With curly braces for multi-line bodies (you must use `return` explicitly):

```js
const describe = (name, age) => {
  const label = `${name} is ${age}`;
  return label.toUpperCase();
};
describe("Alice", 30);
```

Expected output: `'ALICE IS 30'`

<details>
<summary>Python comparison</summary>

Arrow functions are like `lambda` but far more powerful — they can have multi-line bodies with curly braces. They're used everywhere in JS, not just for simple one-liners.

```python
# Single-expression arrows are like lambda:
double = lambda x: x * 2

# Multi-line arrows are like regular def functions:
def describe(name, age):
    label = f"{name} is {age}"
    return label.upper()
```

The key difference: without curly braces, the return is implicit (`x => x * 2`). With curly braces, you need an explicit `return` statement — forgetting it is a common bug.
</details>

### 5. Array methods

```js
const nums = [1, 2, 3, 4, 5];
nums.map(x => x * 2);
```

Expected output: `[2, 4, 6, 8, 10]`

```js
nums.filter(x => x > 3);
```

Expected output: `[4, 5]`

```js
nums.reduce((acc, x) => acc + x, 0);
```

Expected output: `15`

<details>
<summary>Python comparison</summary>

```python
# Python equivalents
[x * 2 for x in nums]         # map
[x for x in nums if x > 3]    # filter
sum(nums)                       # this specific reduce
from functools import reduce
reduce(lambda acc, x: acc + x, nums, 0)  # general reduce
```

JS doesn't have list comprehensions. You chain `.map()`, `.filter()`, and `.reduce()` instead. This is the single biggest syntax shift you'll internalize.
</details>

### 6. typeof operator

```js
typeof "hello";
typeof 42;
typeof true;
typeof undefined;
typeof null;
typeof [];
typeof {};
```

Expected outputs:
```
'string'
'number'
'boolean'
'undefined'
'object'    // Yes, really. This is a famous JS bug.
'object'    // Arrays are objects in JS
'object'
```

<details>
<summary>How to actually check types</summary>

- For null: `x === null`
- For arrays: `Array.isArray(x)`
- For everything else: `typeof x` works fine
- Python's `type()` and `isinstance()` are more consistent, but you get used to this.
</details>

### 7. Truthiness edge cases

```js
Boolean(0);
Boolean("");
Boolean(null);
Boolean(undefined);
Boolean([]);
Boolean({});
Boolean("0");
Boolean(NaN);
```

Expected outputs:
```
false
false
false
false
true     // Empty array is truthy! (Python: bool([]) is False)
true     // Empty object is truthy! (Python: bool({}) is False)
true     // "0" is truthy! (Python: bool("0") is True too, same here)
false    // NaN is falsy
```

<details>
<summary>The rule to memorize</summary>

JS falsy values: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`.

**Everything else is truthy.** Including empty arrays and empty objects. This is the #1 gotcha coming from Python.

To check for an empty array, use `arr.length === 0`, not `!arr`.
</details>

### 8. Equality operators

```js
0 == "";
0 === "";
null == undefined;
null === undefined;
1 == "1";
1 === "1";
```

Expected outputs:
```
true     // loose equality does type coercion
false    // strict equality, no coercion
true     // this is the one useful case of ==
false
true     // coercion again
false
```

<details>
<summary>The rule</summary>

Always use `===` (strict equality). The only exception some codebases allow is `x == null`, which conveniently matches both `null` and `undefined`.

Python's `==` never does type coercion, so there's no equivalent gotcha.

One more thing: `===` compares arrays and objects **by reference**, not by value:

```js
[1, 2, 3] === [1, 2, 3]; // false — two different objects
```

Python's `==` does deep comparison (`[1,2,3] == [1,2,3]` is `True`). JS has no built-in equivalent — use `JSON.stringify()` as a quick hack or a library for deep equality.
</details>

### 9. Object literals

```js
const person = { name: "Alice", age: 30 };
person.name;
person["age"];
Object.keys(person);
Object.entries(person);
```

Expected outputs:
```
'Alice'
30
['name', 'age']
[['name', 'Alice'], ['age', 30]]
```

<details>
<summary>Python comparison</summary>

```python
person = {"name": "Alice", "age": 30}
person["name"]         # No dot notation in Python dicts
person.keys()          # Method on the dict itself
person.items()         # Equivalent to Object.entries()
```

JS object keys don't need quotes (unless they have special characters). You can access them with dot notation (`person.name`) or bracket notation (`person["name"]`). Python dicts only support bracket notation.
</details>

### 10. Destructuring

```js
const { name, age } = { name: "Alice", age: 30 };
name;
age;
```

Expected outputs:
```
'Alice'
30
```

```js
const [first, second, ...rest] = [1, 2, 3, 4, 5];
first;
rest;
```

Expected outputs:
```
1
[3, 4, 5]
```

<details>
<summary>Python comparison</summary>

```python
# Python tuple unpacking is similar for arrays:
first, second, *rest = [1, 2, 3, 4, 5]

# But Python has no equivalent for object destructuring.
# You'd do:
name = person["name"]
age = person["age"]
```

Object destructuring matches by **exact key name** — if the names don't match, you silently get `undefined` (no error):

```js
const { names, ages } = { name: "Alice", age: 30 };
names; // undefined — no key called "names"
```

Destructuring is everywhere in JS/TS code. You'll see it in function parameters, imports, return values, and more.
</details>

### 11. Spread operator

```js
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
[...arr1, ...arr2];
```

Expected output: `[1, 2, 3, 4, 5, 6]`

```js
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };
{ ...obj1, ...obj2 };
```

Expected output: `{ a: 1, b: 3, c: 4 }`

<details>
<summary>Python comparison</summary>

```python
[*list1, *list2]      # Same idea
{**dict1, **dict2}    # Same idea
```

JS uses `...` for both arrays and objects. Python uses `*` for iterables and `**` for dicts.
</details>

### 12. Optional chaining and nullish coalescing

```js
const user = { name: "Alice", address: { city: "NYC" } };
user.address.city;
user.address?.city;
user.phone?.number;
user.phone?.number ?? "N/A";
```

Expected outputs:
```
'NYC'
'NYC'
undefined
'N/A'
```

<details>
<summary>Python comparison</summary>

```python
# No direct equivalent in Python. You'd do:
user.get("address", {}).get("city")           # For dicts
getattr(getattr(user, "address", None), "city", None)  # For objects

# Or just:
user["address"]["city"] if "address" in user else None
```

`?.` (optional chaining) and `??` (nullish coalescing) are two of JS's best features for handling missing data. You'll use them constantly.
</details>

### 13. String methods

```js
"hello world".includes("world");
"hello world".startsWith("hello");
"hello world".split(" ");
"  hello  ".trim();
"hello".padStart(10, ".");
"hello".repeat(3);
```

Expected outputs:
```
true
true
['hello', 'world']
'hello'
'.....hello'
'hellohellohello'
```

<details>
<summary>Python comparison</summary>

```python
"world" in "hello world"          # includes
"hello world".startswith("hello") # startsWith (camelCase in JS)
"hello world".split(" ")          # same!
"  hello  ".strip()               # trim (different name)
"hello".rjust(10, ".")            # padStart
"hello" * 3                       # repeat (operator in Python, method in JS)
```

Method names are camelCase in JS (`startsWith` not `starts_with`). This is true across the entire language and ecosystem.
</details>

### 14. Classes

```js
class Counter {
  constructor(start = 0) {
    this.count = start;
  }

  increment() {
    this.count++;
  }

  toString() {
    return `Counter(${this.count})`;
  }
}

const c = new Counter(5);
c.increment();
c.increment();
c.count;
c.toString();
```

Expected outputs:
```
7
'Counter(7)'
```

<details>
<summary>Python comparison</summary>

```python
class Counter:
    def __init__(self, start=0):
        self.count = start

    def increment(self):
        self.count += 1

    def __str__(self):
        return f"Counter({self.count})"
```

The differences: `constructor` instead of `__init__`, no `self` parameter (use `this` instead), `toString()` instead of `__str__`, and methods don't use the `function` keyword. Inheritance uses `extends` and `super()` just like Python.
</details>

### 15. Conditionals

```js
const score = 85;
if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else {
  console.log("C");
}
```

Expected output: `B`

Ternary expressions work like Python's inline if/else:

```js
const grade = score >= 90 ? "A" : "B";
grade;
```

Expected output: `'B'`

<details>
<summary>Python comparison</summary>

```python
if score >= 90:
    print("A")
elif score >= 80:
    print("B")
else:
    print("C")

grade = "A" if score >= 90 else "B"
```

The differences: curly braces instead of colons/indentation, `else if` instead of `elif`, and parentheses around the condition are optional but common. The ternary syntax (`condition ? a : b`) is more concise than Python's `a if condition else b`.
</details>

### 15. for...of loops

```js
for (const item of [10, 20, 30]) { console.log(item); }
```

Expected output:
```
10
20
30
```

<details>
<summary>Python comparison</summary>

```python
for item in [10, 20, 30]:
    print(item)
```

Python's `for x in` is JS's `for (const x of)`. Don't confuse `for...of` (values) with `for...in` (keys) -- this trips up everyone.

Why `const` in a loop? Each iteration creates a fresh binding — it's not reassigning the same variable. `let` also works, but `const` is preferred since you rarely need to reassign the loop variable.
</details>

### 15. Method chaining

```js
[1, 2, 3]
  .filter(x => x > 1)
  .map(x => x * 10)
  .reduce((sum, x) => sum + x, 0);
```

Expected output: `50`

<details>
<summary>Python comparison</summary>

```python
# Python would use nested comprehensions or separate steps:
nums = [x * 10 for x in [1, 2, 3] if x > 1]
sum(nums)  # 50
```

Method chaining like this is idiomatic JS. You'll chain `.map().filter().reduce()` the way you'd nest list comprehensions in Python, but often more readably.
</details>

## Wrap Up

You've now seen the major syntax differences. The key takeaways:

1. **`const`/`let`** instead of bare variable names
2. **Arrow functions** (`=>`) instead of `lambda` (and used everywhere)
3. **`.map()/.filter()/.reduce()`** instead of list comprehensions
4. **Template literals** (backticks) instead of f-strings
5. **`===`** for equality (always, not `==`)
6. **Empty arrays/objects are truthy** (the biggest gotcha)
7. **Optional chaining** (`?.`) and **nullish coalescing** (`??`) are great
8. **camelCase** everywhere, not snake_case

Next: [Exercise 02: Python to JS Rosetta Stone](./02-python-to-js-rosetta.md)
