# Exercise 02: Python to JS Rosetta Stone

**Time:** ~30 minutes

Translate five small Python programs into JavaScript. This builds muscle memory for the syntax differences you explored in Exercise 01.

## Instructions

- For each snippet, rewrite the Python code in JavaScript
- Run your JS version in Node (`node yourfile.js` or paste into the REPL)
- Verify it produces the same output as the Python version
- Solutions are in `<details>` blocks -- try first before looking

## Snippet 1: String Formatting and Manipulation

Translate this Python code into JavaScript:

```python
def format_experiment_name(model: str, dataset: str, run_id: int) -> str:
    """Create a standardized experiment name."""
    name = f"{model}_{dataset}_run{run_id:03d}"
    return name.upper()

def summarize_text(text: str, max_length: int = 50) -> str:
    """Truncate text with ellipsis if too long."""
    text = text.strip()
    if len(text) <= max_length:
        return text
    return text[:max_length - 3] + "..."

# Test
print(format_experiment_name("bert", "squad", 7))
print(summarize_text("  This is a very long description of an experiment that goes on and on  "))
print(summarize_text("Short text"))
```

Expected output:
```
BERT_SQUAD_RUN007
This is a very long description of an experimen...
Short text
```

<details>
<summary>Hint</summary>

- f-strings become template literals: `` `${variable}` ``
- Zero-padding: JS has `String.prototype.padStart(width, char)`
- `len(s)` becomes `s.length`
- `s.strip()` becomes `s.trim()`
- `s[:n]` becomes `s.slice(0, n)`
- `s.upper()` becomes `s.toUpperCase()`
</details>

<details>
<summary>Solution</summary>

```js
const formatExperimentName = (model, dataset, runId) => {
  const name = `${model}_${dataset}_run${String(runId).padStart(3, "0")}`;
  return name.toUpperCase();
};

const summarizeText = (text, maxLength = 50) => {
  const trimmed = text.trim();
  if (trimmed.length <= maxLength) {
    return trimmed;
  }
  return trimmed.slice(0, maxLength - 3) + "...";
};

// Test
console.log(formatExperimentName("bert", "squad", 7));
console.log(summarizeText("  This is a very long description of an experiment that goes on and on  "));
console.log(summarizeText("Short text"));
```
</details>

## Snippet 2: List Comprehension to Array Methods

Translate this Python code into JavaScript:

```python
scores = [
    {"name": "Alice", "score": 92},
    {"name": "Bob", "score": 78},
    {"name": "Charlie", "score": 85},
    {"name": "Diana", "score": 95},
    {"name": "Eve", "score": 67},
]

# Get names of students who scored above 80, sorted by score descending
honor_roll = [
    s["name"]
    for s in sorted(scores, key=lambda s: s["score"], reverse=True)
    if s["score"] > 80
]

# Compute the average score
avg = sum(s["score"] for s in scores) / len(scores)

# Create a grade summary
grades = {
    s["name"]: "pass" if s["score"] >= 70 else "fail"
    for s in scores
}

print(honor_roll)
print(f"Average: {avg:.1f}")
print(grades)
```

Expected output:
```
['Diana', 'Alice', 'Charlie']
Average: 83.4
{'Diana': 'pass', 'Alice': 'pass', 'Charlie': 'pass', 'Bob': 'pass', 'Eve': 'fail'}
```

<details>
<summary>Hint</summary>

- List comprehension with filter becomes `.filter().map()` (or reversed, depending on order of operations)
- `sorted()` doesn't exist; use `[...arr].sort()` to avoid mutating the original
- Sort comparator takes `(a, b)` and returns a number (negative, zero, or positive)
- Dict comprehension becomes `Object.fromEntries()` with `.map()`
- `sum()` becomes `.reduce()`
- Ternary: `"pass" if x else "fail"` becomes `x ? "pass" : "fail"`
</details>

<details>
<summary>Solution</summary>

```js
const scores = [
  { name: "Alice", score: 92 },
  { name: "Bob", score: 78 },
  { name: "Charlie", score: 85 },
  { name: "Diana", score: 95 },
  { name: "Eve", score: 67 },
];

// Get names of students who scored above 80, sorted by score descending
const honorRoll = [...scores]
  .sort((a, b) => b.score - a.score)
  .filter(s => s.score > 80)
  .map(s => s.name);

// Compute the average score
const avg = scores.reduce((sum, s) => sum + s.score, 0) / scores.length;

// Create a grade summary
const grades = Object.fromEntries(
  scores.map(s => [s.name, s.score >= 70 ? "pass" : "fail"])
);

console.log(honorRoll);
console.log(`Average: ${avg.toFixed(1)}`);
console.log(grades);
```

Note: `Object.fromEntries()` takes an array of `[key, value]` pairs and builds an object -- it's the inverse of `Object.entries()`. This is the JS equivalent of a dict comprehension.
</details>

## Snippet 3: Dict Operations to Object Operations

Translate this Python code into JavaScript:

```python
defaults = {
    "learning_rate": 0.001,
    "batch_size": 32,
    "epochs": 10,
    "optimizer": "adam",
}

overrides = {
    "batch_size": 64,
    "epochs": 20,
    "dropout": 0.5,
}

# Merge with overrides taking precedence
config = {**defaults, **overrides}

# Remove a key
del config["dropout"]

# Check for key existence and safe access
has_lr = "learning_rate" in config
warmup = config.get("warmup_steps", 0)

# Iterate over key-value pairs
for key, value in sorted(config.items()):
    print(f"  {key}: {value}")

print(f"Has learning_rate: {has_lr}")
print(f"Warmup steps: {warmup}")
print(f"Keys: {sorted(config.keys())}")
```

Expected output:
```
  batch_size: 64
  epochs: 20
  learning_rate: 0.001
  optimizer: adam
Has learning_rate: true
Warmup steps: 0
Keys: ['batch_size', 'epochs', 'learning_rate', 'optimizer']
```

<details>
<summary>Hint</summary>

- `{**a, **b}` becomes `{ ...a, ...b }`
- `del d[k]` becomes `delete obj[k]` (or use destructuring to omit a key)
- `"key" in dict` works the same: `"key" in obj`
- `dict.get(key, default)` becomes `obj[key] ?? default` (nullish coalescing)
- `dict.items()` becomes `Object.entries(obj)`
- `sorted()` becomes `[...arr].sort()`
- Python booleans are `True`/`False`; JS uses `true`/`false`
</details>

<details>
<summary>Solution</summary>

```js
const defaults = {
  learning_rate: 0.001,
  batch_size: 32,
  epochs: 10,
  optimizer: "adam",
};

const overrides = {
  batch_size: 64,
  epochs: 20,
  dropout: 0.5,
};

// Merge with overrides taking precedence
const config = { ...defaults, ...overrides };

// Remove a key
delete config.dropout;

// Check for key existence and safe access
const hasLr = "learning_rate" in config;
const warmup = config.warmup_steps ?? 0;

// Iterate over key-value pairs
for (const [key, value] of Object.entries(config).sort((a, b) => a[0].localeCompare(b[0]))) {
  console.log(`  ${key}: ${value}`);
}

console.log(`Has learning_rate: ${hasLr}`);
console.log(`Warmup steps: ${warmup}`);
console.log(`Keys: ${JSON.stringify(Object.keys(config).sort())}`);
```

Notes:
- JS `sort()` on strings works lexicographically by default (unlike numbers, where you need a comparator).
- `JSON.stringify()` is used to print the array in bracket notation. `console.log` would also show it, but formatted differently.
- `localeCompare` is the standard way to sort strings by comparison function.
</details>

## Snippet 4: Function with Default Args Returning a Dict

Translate this Python code into JavaScript:

```python
from datetime import datetime

def create_experiment(
    name: str,
    model_type: str,
    hyperparameters: dict | None = None,
    tags: list[str] | None = None,
) -> dict:
    """Create an experiment configuration dictionary."""
    return {
        "name": name,
        "model_type": model_type,
        "hyperparameters": hyperparameters or {"learning_rate": 0.001},
        "tags": tags or [],
        "created_at": datetime.now().isoformat(),
        "status": "created",
    }

# Test
exp1 = create_experiment("bert-finetuning", "transformer")
print(f"Name: {exp1['name']}")
print(f"Hyperparameters: {exp1['hyperparameters']}")
print(f"Tags: {exp1['tags']}")
print(f"Status: {exp1['status']}")

exp2 = create_experiment(
    "cnn-classifier",
    "cnn",
    hyperparameters={"lr": 0.01, "dropout": 0.3},
    tags=["vision", "production"],
)
print(f"\nName: {exp2['name']}")
print(f"Tags: {exp2['tags']}")
```

Expected output (timestamp will differ):
```
Name: bert-finetuning
Hyperparameters: {'learning_rate': 0.001}
Tags: []
Status: created

Name: cnn-classifier
Tags: ['vision', 'production']
```

<details>
<summary>Hint</summary>

- `dict | None = None` default args work the same way in JS: `param = null`
- `hyperparameters or {"learning_rate": 0.001}` uses the `||` operator in JS -- or better, `??` for nullish coalescing
- `datetime.now().isoformat()` becomes `new Date().toISOString()`
- JS doesn't have keyword arguments. You pass them positionally, or use an options object (common pattern, covered in Module 01).
- `console.log` prints objects nicely on its own, but inside template literals you need `JSON.stringify()` to avoid getting `[object Object]`
</details>

<details>
<summary>Solution</summary>

```js
const createExperiment = (name, modelType, hyperparameters = null, tags = null) => {
  return {
    name,                // Shorthand: { name } is the same as { name: name }
    modelType,
    hyperparameters: hyperparameters ?? { learning_rate: 0.001 },
    tags: tags ?? [],
    createdAt: new Date().toISOString(),
    status: "created",
  };
};

// Test
const exp1 = createExperiment("bert-finetuning", "transformer");
console.log(`Name: ${exp1.name}`);
console.log(`Hyperparameters: ${JSON.stringify(exp1.hyperparameters)}`);
console.log(`Tags: ${JSON.stringify(exp1.tags)}`);
console.log(`Status: ${exp1.status}`);

const exp2 = createExperiment(
  "cnn-classifier",
  "cnn",
  { lr: 0.01, dropout: 0.3 },
  ["vision", "production"]
);
console.log(`\nName: ${exp2.name}`);
console.log(`Tags: ${JSON.stringify(exp2.tags)}`);
```

Key differences:
- **Property shorthand**: `{ name }` instead of `{ name: name }` when the variable name matches the key.
- **`??` vs `||`**: Use `??` (nullish coalescing) here because `||` would also replace falsy values like `0` or `""`.
- **No keyword arguments**: JS doesn't support `func(key=value)` syntax. For functions with many optional params, the idiomatic pattern is to accept an options object: `createExperiment("name", { tags: [...] })`. You'll see this pattern everywhere.
</details>

## Snippet 5: Class with Inheritance

Translate this Python code into JavaScript:

```python
class Metric:
    def __init__(self, name: str, values: list[float] | None = None):
        self.name = name
        self.values = values or []

    def add(self, value: float) -> None:
        self.values.append(value)

    def mean(self) -> float:
        if not self.values:
            return 0.0
        return sum(self.values) / len(self.values)

    def __str__(self) -> str:
        return f"{self.name}: mean={self.mean():.4f} (n={len(self.values)})"


class BoundedMetric(Metric):
    """A metric that only accepts values within a range."""

    def __init__(self, name: str, min_val: float = 0.0, max_val: float = 1.0):
        super().__init__(name)
        self.min_val = min_val
        self.max_val = max_val

    def add(self, value: float) -> None:
        if not (self.min_val <= value <= self.max_val):
            raise ValueError(
                f"Value {value} out of range [{self.min_val}, {self.max_val}]"
            )
        super().add(value)


# Test
accuracy = Metric("accuracy")
accuracy.add(0.85)
accuracy.add(0.90)
accuracy.add(0.88)
print(accuracy)

loss = BoundedMetric("loss", 0.0, 10.0)
loss.add(2.5)
loss.add(1.8)
print(loss)

try:
    loss.add(-1.0)
except ValueError as e:
    print(f"Caught: {e}")
```

Expected output:
```
accuracy: mean=0.8767 (n=3)
loss: mean=2.1500 (n=2)
Caught: Value -1 out of range [0, 10]
```

<details>
<summary>Hint</summary>

- `constructor` instead of `__init__`, `this` instead of `self`
- `.append(value)` becomes `.push(value)`
- `class Child extends Parent { ... }` instead of `class Child(Parent):`
- Call `super(args)` in the constructor to invoke the parent's constructor (like `super().__init__(args)`)
- Call `super.methodName(args)` to invoke a parent method (like `super().methodName(args)`)
- `raise ValueError(...)` becomes `throw new Error(...)`
- `toString()` instead of `__str__`
- `sum()` becomes `.reduce((a, b) => a + b, 0)`
- `len()` becomes `.length`
- `f"{value:.4f}"` becomes `value.toFixed(4)`
- JS doesn't support chained comparisons like `a <= x <= b`. Use `x < a || x > b` (or `!(x >= a && x <= b)`)
- `try/except` becomes `try { ... } catch (e) { ... }` — and `e.message` instead of `str(e)`
- Empty arrays are truthy in JS, so use `this.values.length === 0` instead of `not self.values`
</details>

<details>
<summary>Solution</summary>

```js
class Metric {
  constructor(name, values = null) {
    this.name = name;
    this.values = values ?? [];
  }

  add(value) {
    this.values.push(value);
  }

  mean() {
    if (this.values.length === 0) {
      return 0.0;
    }
    return this.values.reduce((sum, v) => sum + v, 0) / this.values.length;
  }

  toString() {
    return `${this.name}: mean=${this.mean().toFixed(4)} (n=${this.values.length})`;
  }
}

class BoundedMetric extends Metric {
  constructor(name, minVal = 0.0, maxVal = 1.0) {
    super(name);       // Must call super() before using 'this'
    this.minVal = minVal;
    this.maxVal = maxVal;
  }

  add(value) {
    if (value < this.minVal || value > this.maxVal) {
      throw new Error(
        `Value ${value} out of range [${this.minVal}, ${this.maxVal}]`
      );
    }
    super.add(value);  // Call parent's add method
  }
}

// Test
const accuracy = new Metric("accuracy");
accuracy.add(0.85);
accuracy.add(0.90);
accuracy.add(0.88);
console.log(accuracy.toString());  // Or: console.log(`${accuracy}`)

const loss = new BoundedMetric("loss", 0.0, 10.0);
loss.add(2.5);
loss.add(1.8);
console.log(loss.toString());

try {
  loss.add(-1.0);
} catch (e) {
  console.log(`Caught: ${e.message}`);
}
```

Note: `new Metric()` is required to instantiate — omitting `new` is a bug.
- **`toString()`** is called automatically by template literals (`` `${accuracy}` ``), similar to Python's `__str__`.
- **`!this.values`** would be truthy even when empty -- must check `.length === 0`
</details>

## Wrap Up

You've now translated five common Python patterns into JavaScript. The patterns to internalize:

1. **List comprehensions** become chains of `.filter()`, `.map()`, `.reduce()`
2. **Dict comprehensions** become `Object.fromEntries(arr.map(...))`
3. **f-strings** become template literals with backticks
4. **Classes** are nearly identical, just different keywords
5. **Error handling** uses `try/catch` and `throw new Error()`

Next: [Module 01: Language Fundamentals](../../01-language-fundamentals/README.md)
