# Exercise 01: Type Annotations

Add full TypeScript type annotations to 10 untyped JavaScript functions. This exercise builds muscle memory for the most common type annotation patterns.

**Time estimate:** 20-30 minutes

## Setup

Create a file called `type-annotations.ts` and copy the untyped functions below into it. Add type annotations to every parameter, return type, and any variables where the type isn't obvious.

Then run:

```bash
npx tsc --strict --noEmit type-annotations.ts
```

Fix any errors until it compiles cleanly.

---

## The Functions

Copy these 10 untyped functions into your `.ts` file and add types to each one.

```javascript
// 1. Basic types
function greet(name, age) {
  return `Hello, ${name}! You are ${age} years old.`;
}

// 2. Array parameter and return
function doubleAll(numbers) {
  return numbers.map(n => n * 2);
}

// 3. Object parameter
function formatUser(user) {
  return `${user.name} (${user.email})`;
}

// 4. Optional parameter
function repeat(text, times) {
  return text.repeat(times ?? 1);
}

// 5. Union type parameter
function stringify(value) {
  if (typeof value === "string") {
    return value;
  }
  return String(value);
}

// 6. Callback parameter
function processItems(items, callback) {
  return items.map(callback);
}

// 7. Object return type
function createPoint(x, y) {
  return { x, y, distanceFromOrigin: Math.sqrt(x * x + y * y) };
}

// 8. Array of objects
function getTopScorers(students, threshold) {
  return students
    .filter(s => s.score >= threshold)
    .map(s => s.name);
}

// 9. Function with multiple return types
function safeDivide(a, b) {
  if (b === 0) {
    return null;
  }
  return a / b;
}

// 10. Complex callback with index
function transformWithIndex(items, transform) {
  return items.map((item, index) => transform(item, index));
}
```

---

## Guidance

For each function, think about:

1. **Parameters:** What type is each parameter? Is it a primitive, an object, an array, a function?
2. **Return type:** What does the function return? Can it return multiple types (union)?
3. **Object shapes:** When a function takes or returns an object, define an `interface` or inline type for its shape.
4. **Callbacks:** Functions passed as parameters need their own parameter and return type annotations.

### Python Comparison

If you were annotating these in Python, you'd write:

```python
def greet(name: str, age: int) -> str: ...
def double_all(numbers: list[int]) -> list[int]: ...
```

TypeScript syntax is very similar:

```typescript
function greet(name: string, age: number): string { ... }
function doubleAll(numbers: number[]): number[] { ... }
```

---

## Solution

<details>
<summary>Hints</summary>

- Function 3 needs an inline object type or a defined interface for the user parameter.
- Function 4: use `?` to mark the optional parameter. Check what `times ?? 1` implies about the type.
- Function 6: the callback takes a single item and returns something. Think about how to type a generic callback -- or just use `string` for the specific case.
- Function 8 needs an interface for the student objects.
- Function 9 returns either `number` or `null` -- that's a union type.
- Function 10: the transform callback takes both the item and the index.

</details>

<details>
<summary>Full Solution</summary>

```typescript
// 1. Basic types
function greet(name: string, age: number): string {
  return `Hello, ${name}! You are ${age} years old.`;
}

// 2. Array parameter and return
function doubleAll(numbers: number[]): number[] {
  return numbers.map(n => n * 2);
}

// 3. Object parameter
interface User {
  name: string;
  email: string;
}

function formatUser(user: User): string {
  return `${user.name} (${user.email})`;
}

// 4. Optional parameter
function repeat(text: string, times?: number): string {
  return text.repeat(times ?? 1);
}

// 5. Union type parameter
function stringify(value: string | number | boolean): string {
  if (typeof value === "string") {
    return value;
  }
  return String(value);
}

// 6. Callback parameter
function processItems(items: string[], callback: (item: string) => string): string[] {
  return items.map(callback);
}

// 7. Object return type
interface Point {
  x: number;
  y: number;
  distanceFromOrigin: number;
}

function createPoint(x: number, y: number): Point {
  return { x, y, distanceFromOrigin: Math.sqrt(x * x + y * y) };
}

// 8. Array of objects
interface Student {
  name: string;
  score: number;
}

function getTopScorers(students: Student[], threshold: number): string[] {
  return students
    .filter(s => s.score >= threshold)
    .map(s => s.name);
}

// 9. Function with multiple return types
function safeDivide(a: number, b: number): number | null {
  if (b === 0) {
    return null;
  }
  return a / b;
}

// 10. Complex callback with index
function transformWithIndex(
  items: string[],
  transform: (item: string, index: number) => string
): string[] {
  return items.map((item, index) => transform(item, index));
}
```

**Note:** Functions 6 and 10 use `string` for concreteness. In real code, you'd likely make them generic:

```typescript
function processItems<T, U>(items: T[], callback: (item: T) => U): U[] {
  return items.map(callback);
}

function transformWithIndex<T, U>(
  items: T[],
  transform: (item: T, index: number) => U
): U[] {
  return items.map((item, index) => transform(item, index));
}
```

Generics are covered in exercise 03.

</details>

---

## Verification

After adding types, run:

```bash
npx tsc --strict --noEmit type-annotations.ts
```

If there are no errors, you're done. Try calling each function with wrong argument types to see the compiler catch mistakes:

```typescript
greet(42, "Alice");          // Error: types are swapped
doubleAll(["a", "b", "c"]); // Error: string[] is not number[]
safeDivide("10", 2);        // Error: string is not number
```
