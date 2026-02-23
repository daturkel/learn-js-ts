# Exercise: Functions and Array Methods

Rewrite a Python data processing pipeline using JavaScript array methods.

## Setup

Create `functions.js` and run with `node functions.js`.

## The Python Version

Here's a Python script that processes ML experiment results:

```python
experiments = [
    {"name": "bert-base", "model": "bert", "accuracy": 0.891, "training_hours": 2.5, "status": "completed"},
    {"name": "bert-large", "model": "bert", "accuracy": 0.923, "training_hours": 8.0, "status": "completed"},
    {"name": "gpt2-small", "model": "gpt2", "accuracy": 0.845, "training_hours": 1.5, "status": "completed"},
    {"name": "gpt2-medium", "model": "gpt2", "accuracy": 0.901, "training_hours": 5.0, "status": "completed"},
    {"name": "t5-base", "model": "t5", "accuracy": 0.912, "training_hours": 4.0, "status": "completed"},
    {"name": "t5-failed", "model": "t5", "accuracy": 0.0, "training_hours": 0.5, "status": "failed"},
    {"name": "roberta", "model": "roberta", "accuracy": 0.935, "training_hours": 6.0, "status": "completed"},
]

# Task 1: Get completed experiments with accuracy > 0.9
high_performers = [e for e in experiments if e["status"] == "completed" and e["accuracy"] > 0.9]
# => bert-large, gpt2-medium, t5-base, roberta

# Task 2: Extract just names and accuracies
summaries = [{"name": e["name"], "accuracy": e["accuracy"]} for e in high_performers]

# Task 3: Sort by accuracy, descending
summaries.sort(key=lambda e: e["accuracy"], reverse=True)

# Task 4: Compute average accuracy of completed experiments
completed = [e for e in experiments if e["status"] == "completed"]
avg_accuracy = sum(e["accuracy"] for e in completed) / len(completed)

# Task 5: Group experiments by model
from collections import defaultdict
by_model = defaultdict(list)
for e in experiments:
    by_model[e["model"]].append(e["name"])

# Task 6: Find the experiment with the longest training time
longest = max(experiments, key=lambda e: e["training_hours"])
```

## Tasks

Rewrite each of the 6 tasks in JavaScript using array methods (`filter`, `map`, `sort`, `reduce`, `find`, etc.).

Start by defining the data:

```javascript
const experiments = [
  { name: "bert-base", model: "bert", accuracy: 0.891, trainingHours: 2.5, status: "completed" },
  { name: "bert-large", model: "bert", accuracy: 0.923, trainingHours: 8.0, status: "completed" },
  { name: "gpt2-small", model: "gpt2", accuracy: 0.845, trainingHours: 1.5, status: "completed" },
  { name: "gpt2-medium", model: "gpt2", accuracy: 0.901, trainingHours: 5.0, status: "completed" },
  { name: "t5-base", model: "t5", accuracy: 0.912, trainingHours: 4.0, status: "completed" },
  { name: "t5-failed", model: "t5", accuracy: 0.0, trainingHours: 0.5, status: "failed" },
  { name: "roberta", model: "roberta", accuracy: 0.935, trainingHours: 6.0, status: "completed" },
];
```

Then implement:
1. Filter to completed experiments with accuracy > 0.9
2. Map to just `{ name, accuracy }` objects
3. Sort by accuracy descending (without mutating the original)
4. Compute average accuracy of completed experiments
5. Group experiments by model (into an object like `{ bert: [...], gpt2: [...] }`)
6. Find the experiment with the longest training time

<details>
<summary>Hint: Task 1</summary>

Use `.filter()` with two conditions joined by `&&`:
```javascript
experiments.filter(e => condition1 && condition2)
```
</details>

<details>
<summary>Hint: Task 3</summary>

`.sort()` mutates the array. To avoid that, spread into a new array first:
```javascript
[...array].sort((a, b) => b.accuracy - a.accuracy)
```

The comparator returns negative (a first), zero (equal), or positive (b first).
</details>

<details>
<summary>Hint: Task 5</summary>

Use `.reduce()` to build up an object. The accumulator starts as `{}`:
```javascript
array.reduce((groups, item) => {
  // Add item to the right group
  return groups;
}, {})
```

To use a variable as an object key, use bracket notation in the object literal:
```javascript
const key = item.model;
{ [key]: value }   // uses the VALUE of key, not the literal string "key"
```
</details>

<details>
<summary>Solution</summary>

```javascript
const experiments = [
  { name: "bert-base", model: "bert", accuracy: 0.891, trainingHours: 2.5, status: "completed" },
  { name: "bert-large", model: "bert", accuracy: 0.923, trainingHours: 8.0, status: "completed" },
  { name: "gpt2-small", model: "gpt2", accuracy: 0.845, trainingHours: 1.5, status: "completed" },
  { name: "gpt2-medium", model: "gpt2", accuracy: 0.901, trainingHours: 5.0, status: "completed" },
  { name: "t5-base", model: "t5", accuracy: 0.912, trainingHours: 4.0, status: "completed" },
  { name: "t5-failed", model: "t5", accuracy: 0.0, trainingHours: 0.5, status: "failed" },
  { name: "roberta", model: "roberta", accuracy: 0.935, trainingHours: 6.0, status: "completed" },
];

// Task 1: Filter — completed with accuracy > 0.9
const highPerformers = experiments.filter(
  e => e.status === "completed" && e.accuracy > 0.9
);
console.log("High performers:", highPerformers.map(e => e.name));
// ["bert-large", "gpt2-medium", "t5-base", "roberta"]

// Task 2: Map — extract name and accuracy
const summaries = highPerformers.map(e => ({ name: e.name, accuracy: e.accuracy }));
// Note: wrapping in () is needed so JS doesn't think { } is a code block
console.log("Summaries:", summaries);

// Task 3: Sort — by accuracy descending (non-mutating)
const sorted = [...summaries].sort((a, b) => b.accuracy - a.accuracy);
console.log("Sorted:", sorted);
// [roberta 0.935, bert-large 0.923, t5-base 0.912, gpt2-medium 0.901]

// Task 4: Reduce — average accuracy of completed
const completed = experiments.filter(e => e.status === "completed");
const avgAccuracy = completed.reduce((sum, e) => sum + e.accuracy, 0) / completed.length;
console.log("Average accuracy:", avgAccuracy.toFixed(3));
// 0.901

// Task 5: Reduce — group by model
const byModel = experiments.reduce((groups, e) => {
  const existing = groups[e.model] ?? [];
  return { ...groups, [e.model]: [...existing, e.name] };
}, {});
console.log("By model:", byModel);
// { bert: ["bert-base", "bert-large"], gpt2: ["gpt2-small", "gpt2-medium"], ... }

// Alternative Task 5 using Object.groupBy (modern JS):
// const byModel = Object.groupBy(experiments, e => e.model);

// Task 6: Reduce — find longest training time
const longest = experiments.reduce((max, e) =>
  e.trainingHours > max.trainingHours ? e : max
);
console.log("Longest:", longest.name, longest.trainingHours);
// "bert-large" 8.0
```

### Key Differences from Python

- **List comprehension → `.filter().map()`** — you chain methods instead of using `[... for ... in ... if ...]`
- **`sorted(key=lambda)`  → `[...arr].sort((a, b) => ...)`** — JS sort takes a comparator (returns number), not a key function
- **`defaultdict` → `.reduce()`** — no built-in equivalent; reduce builds up the result
- **`max(key=lambda)` → `.reduce()`** — no built-in max with key function
- **Returning an object from arrow function needs `()`** — `x => ({ key: val })` not `x => { key: val }`
- **Computed property names** — `{ [e.model]: value }` uses the *value* of `e.model` as the key. Without brackets, you'd get a key literally called `"e.model"`. In Python dicts, keys are always expressions so this isn't needed.
</details>

## Stretch Goal

Chain Tasks 1, 2, and 3 into a single expression (no intermediate variables):

<details>
<summary>Solution</summary>

```javascript
const result = experiments
  .filter(e => e.status === "completed" && e.accuracy > 0.9)
  .map(e => ({ name: e.name, accuracy: e.accuracy }))
  .sort((a, b) => b.accuracy - a.accuracy);
```

This chaining pattern is idiomatic JS. Each method returns a new array that the next method operates on.
</details>

## Bonus: Closures

Write a function `makeThresholdFilter` that takes a minimum accuracy and returns a new function that filters an experiment array by that threshold:

```javascript
const filterAbove90 = makeThresholdFilter(0.9);
const filterAbove85 = makeThresholdFilter(0.85);

console.log(filterAbove90(experiments).map(e => e.name));
// ["bert-large", "gpt2-medium", "t5-base", "roberta"]

console.log(filterAbove85(experiments).map(e => e.name));
// ["bert-base", "bert-large", "gpt2-medium", "t5-base", "roberta"]
```

This is a closure because the returned function "closes over" the `minAccuracy` parameter — it remembers the value even after `makeThresholdFilter` has returned.

<details>
<summary>Solution</summary>

```javascript
const makeThresholdFilter = (minAccuracy) => {
  return (experiments) =>
    experiments.filter(e => e.status === "completed" && e.accuracy > minAccuracy);
};
```

This pattern — a function that returns a configured function — is common in JS. You'll see it in middleware, event handlers, and React hooks.
</details>

---

**Next:** [Exercise 3: Objects and Classes →](/01-language-fundamentals/exercises/03-objects-and-classes.md)
