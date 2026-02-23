# Exercise: Types and Variables

Predict the output of each snippet, then verify in the Node REPL (`node`).

## Instructions

For each snippet:
1. Read the code and write down what you think it outputs
2. Run it in `node` to check
3. If you were wrong, read the explanation

## Snippets

### 1. const with objects

```javascript
const person = { name: "Alice" };
person.name = "Bob";
console.log(person.name);
```

<details>
<summary>Answer</summary>

`"Bob"` — `const` prevents reassigning the variable, not mutating the value. The object's properties can still be changed.
</details>

### 2. const with arrays

```javascript
const items = [1, 2, 3];
items.push(4);
console.log(items.length);
```

<details>
<summary>Answer</summary>

`4` — Same principle. `const` means the binding is constant, not the value. You can push, pop, and mutate the array.
</details>

### 3. Truthiness — empty array

```javascript
if ([]) {
  console.log("truthy");
} else {
  console.log("falsy");
}
```

<details>
<summary>Answer</summary>

`"truthy"` — Empty arrays are truthy in JavaScript! This is different from Python where `bool([])` is `False`. To check if an array is empty, use `if (arr.length)`.
</details>

### 4. Truthiness — empty object

```javascript
if ({}) {
  console.log("truthy");
} else {
  console.log("falsy");
}
```

<details>
<summary>Answer</summary>

`"truthy"` — Empty objects are also truthy. To check if an object is "empty", use `if (Object.keys(obj).length)`.
</details>

### 5. Truthiness — zero and empty string

```javascript
console.log(Boolean(0));
console.log(Boolean(""));
console.log(Boolean("0"));
console.log(Boolean(" "));
```

<details>
<summary>Answer</summary>

```
false
false
true
true
```

`0` and `""` are falsy. But `"0"` (non-empty string) and `" "` (string with a space) are truthy. Python would also consider `"0"` truthy, so that matches — but it surprises people.
</details>

### 6. == vs ===

```javascript
console.log(0 == "");
console.log(0 === "");
console.log(null == undefined);
console.log(null === undefined);
```

<details>
<summary>Answer</summary>

```
true
false
true
false
```

`==` does type coercion. `===` does strict comparison. The `null == undefined` being `true` is actually the one useful case of `==` — it lets you check for both with one comparison.
</details>

### 7. typeof quirks

```javascript
console.log(typeof null);
console.log(typeof []);
console.log(typeof {});
console.log(typeof undefined);
```

<details>
<summary>Answer</summary>

```
"object"
"object"
"object"
"undefined"
```

`typeof null === "object"` is a famous bug from 1995 that can never be fixed (would break the web). Arrays are also `"object"` — use `Array.isArray()` to check.
</details>

### 8. let vs var scoping

```javascript
for (var i = 0; i < 3; i++) {}
console.log(i);

for (let j = 0; j < 3; j++) {}
console.log(j);
```

<details>
<summary>Answer</summary>

First `console.log(i)` prints `3` — `var` leaks out of the for loop scope.

Second `console.log(j)` throws `ReferenceError: j is not defined` — `let` is block-scoped and doesn't exist outside the loop.

This is why you never use `var`.
</details>

### 9. undefined vs null

```javascript
let x;
let y = null;
console.log(x);
console.log(y);
console.log(x == y);
console.log(x === y);
```

<details>
<summary>Answer</summary>

```
undefined
null
true
false
```

Uninitialized variables are `undefined`. `null` is an explicit assignment. They're `==` equal but not `===` equal.
</details>

### 10. NaN

```javascript
console.log(typeof NaN);
console.log(NaN === NaN);
console.log(Number.isNaN(NaN));
console.log(Number.isNaN("hello"));
```

<details>
<summary>Answer</summary>

```
"number"
false
true
false
```

`NaN` is technically a "number" type. `NaN` is not equal to itself (!) — the only value in JS where `x !== x`. Use `Number.isNaN()` to check.
</details>

### 11. String to number coercion

```javascript
console.log("5" + 3);
console.log("5" - 3);
console.log("5" * 2);
console.log(+"5");
```

<details>
<summary>Answer</summary>

```
"53"
2
10
5
```

`+` with a string triggers string concatenation. `-` and `*` trigger numeric coercion. Unary `+` converts to number. This is a major footgun — always be explicit about types.
</details>

### 12. Object property access

```javascript
const obj = { 1: "one", "2": "two" };
console.log(obj[1]);
console.log(obj["1"]);
console.log(obj[2]);
```

<details>
<summary>Answer</summary>

```
"one"
"one"
"two"
```

Object keys are always strings (or Symbols). The number `1` is coerced to string `"1"`. So `obj[1]` and `obj["1"]` access the same property.
</details>

### 13. Array holes

```javascript
const arr = [1, , 3];
console.log(arr.length);
console.log(arr[1]);
```

<details>
<summary>Answer</summary>

```
3
undefined
```

Skipping an element with `,,` creates a "hole" — accessing it returns `undefined`. The length still counts it. Avoid this pattern.
</details>

### 14. String comparison

```javascript
console.log("apple" < "banana");
console.log("10" < "9");
console.log(10 < 9);
```

<details>
<summary>Answer</summary>

```
true
true
false
```

String comparison is lexicographic (character by character). `"10" < "9"` because `"1" < "9"`. Numeric comparison works as expected. Same as Python actually, but easy to forget when values come from user input as strings.
</details>

### 15. Destructuring with defaults

```javascript
const { a = 1, b = 2, c = 3 } = { a: 10, c: undefined };
console.log(a, b, c);
```

<details>
<summary>Answer</summary>

```
10 2 3
```

`a` gets the provided value `10`. `b` is missing from the object, so gets default `2`. `c` is explicitly `undefined`, which also triggers the default `3`. Note: `null` would NOT trigger the default.
</details>

## Score Yourself

- **12-15 correct:** You already think in JS. Move on quickly.
- **8-11 correct:** Good instincts, just need to internalize the gotchas.
- **Under 8:** Re-read the relevant sections of the [module README](../README.md), then try again.

---

**Next:** [Exercise 2: Functions and Array Methods →](/01-language-fundamentals/exercises/02-functions-and-closures.md)
