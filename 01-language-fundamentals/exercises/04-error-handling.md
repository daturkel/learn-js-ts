# Exercise: Error Handling

Write a config parser that validates JSON input and returns typed results.

## Goal

Practice JavaScript's `try/catch` error handling by building a function that:
1. Parses a JSON string
2. Validates that required fields exist and have correct types
3. Returns the validated config or throws descriptive errors

## The Config Format

Your parser should accept JSON like this:

```json
{
  "model": "bert-base",
  "learningRate": 0.001,
  "batchSize": 32,
  "epochs": 10,
  "optimizer": "adam",
  "notes": "First attempt"
}
```

Rules:
- `model` — required, must be a non-empty string
- `learningRate` — required, must be a positive number
- `batchSize` — required, must be a positive integer
- `epochs` — required, must be a positive integer
- `optimizer` — optional, defaults to `"adam"`, must be one of: `"adam"`, `"sgd"`, `"adamw"`
- `notes` — optional, defaults to `""`

## Your Task

### Step 1: Write `parseConfig(jsonString)`

The function should:
- Parse the JSON string (handle invalid JSON gracefully)
- Validate each field
- Return a clean config object with defaults applied
- Throw `Error` with descriptive messages for validation failures

### Step 2: Write `loadConfig(jsonString)`

A wrapper that calls `parseConfig` and handles errors:
- If parsing succeeds, return `{ ok: true, config: ... }`
- If parsing fails, return `{ ok: false, error: "..." }`
- Never throws — always returns a result

### Step 3: Test with these inputs

```javascript
// Should succeed
loadConfig('{"model":"bert","learningRate":0.001,"batchSize":32,"epochs":10}');

// Should fail: invalid JSON
loadConfig('not json at all');

// Should fail: missing required field
loadConfig('{"model":"bert","batchSize":32,"epochs":10}');

// Should fail: wrong type
loadConfig('{"model":"bert","learningRate":"fast","batchSize":32,"epochs":10}');

// Should fail: invalid optimizer
loadConfig('{"model":"bert","learningRate":0.001,"batchSize":32,"epochs":10,"optimizer":"momentum"}');

// Should succeed with defaults
loadConfig('{"model":"bert","learningRate":0.001,"batchSize":32,"epochs":5}');
```

<details>
<summary>Hint: Catching JSON parse errors</summary>

```javascript
try {
  const data = JSON.parse(jsonString);
} catch (error) {
  // JSON.parse throws SyntaxError on invalid JSON
  throw new Error(`Invalid JSON: ${error.message}`);
}
```
</details>

<details>
<summary>Hint: Checking for integers</summary>

JavaScript has no integer type. Check with `Number.isInteger()`:
```javascript
Number.isInteger(32);    // true
Number.isInteger(32.5);  // false
Number.isInteger("32");  // false
```
</details>

<details>
<summary>Solution</summary>

```javascript
function parseConfig(jsonString) {
  // Step 1: Parse JSON
  let raw;
  try {
    raw = JSON.parse(jsonString);
  } catch (error) {
    throw new Error(`Invalid JSON: ${error.message}`);
  }

  // Ensure it's an object
  if (typeof raw !== "object" || raw === null || Array.isArray(raw)) {
    throw new Error("Config must be a JSON object");
  }

  // Step 2: Validate required fields
  // model
  if (typeof raw.model !== "string" || raw.model.length === 0) {
    throw new Error("'model' is required and must be a non-empty string");
  }

  // learningRate
  if (typeof raw.learningRate !== "number" || raw.learningRate <= 0) {
    throw new Error("'learningRate' is required and must be a positive number");
  }

  // batchSize
  if (!Number.isInteger(raw.batchSize) || raw.batchSize <= 0) {
    throw new Error("'batchSize' is required and must be a positive integer");
  }

  // epochs
  if (!Number.isInteger(raw.epochs) || raw.epochs <= 0) {
    throw new Error("'epochs' is required and must be a positive integer");
  }

  // Step 3: Validate optional fields with defaults
  const validOptimizers = ["adam", "sgd", "adamw"];
  const optimizer = raw.optimizer ?? "adam";
  if (!validOptimizers.includes(optimizer)) {
    throw new Error(`'optimizer' must be one of: ${validOptimizers.join(", ")}`);
  }

  const notes = raw.notes ?? "";
  if (typeof notes !== "string") {
    throw new Error("'notes' must be a string");
  }

  // Return clean config
  return {
    model: raw.model,
    learningRate: raw.learningRate,
    batchSize: raw.batchSize,
    epochs: raw.epochs,
    optimizer,
    notes,
  };
}

function loadConfig(jsonString) {
  try {
    const config = parseConfig(jsonString);
    return { ok: true, config };
  } catch (error) {
    return { ok: false, error: error.message };
  }
}

// Test cases
console.log(loadConfig('{"model":"bert","learningRate":0.001,"batchSize":32,"epochs":10}'));
// { ok: true, config: { model: "bert", learningRate: 0.001, ... optimizer: "adam", notes: "" } }

console.log(loadConfig('not json'));
// { ok: false, error: "Invalid JSON: Unexpected token ..." }

console.log(loadConfig('{"model":"bert","batchSize":32,"epochs":10}'));
// { ok: false, error: "'learningRate' is required and must be a positive number" }

console.log(loadConfig('{"model":"bert","learningRate":"fast","batchSize":32,"epochs":10}'));
// { ok: false, error: "'learningRate' is required and must be a positive number" }

console.log(loadConfig('{"model":"bert","learningRate":0.001,"batchSize":32,"epochs":10,"optimizer":"momentum"}'));
// { ok: false, error: "'optimizer' must be one of: adam, sgd, adamw" }

console.log(loadConfig('{"model":"bert","learningRate":0.001,"batchSize":32,"epochs":5}'));
// { ok: true, config: { ... optimizer: "adam", notes: "" } }
```

### Comparison to Python

```python
import json

def parse_config(json_string: str) -> dict:
    try:
        raw = json.loads(json_string)
    except json.JSONDecodeError as e:
        raise ValueError(f"Invalid JSON: {e}")

    if not isinstance(raw, dict):
        raise ValueError("Config must be a JSON object")

    # In Python you'd probably use pydantic for this:
    # class Config(BaseModel):
    #     model: str
    #     learning_rate: float = Field(gt=0)
    #     ...
```

Key differences:
- Python has `json.JSONDecodeError`; JS has `SyntaxError` from `JSON.parse`
- Python would likely use Pydantic for validation; JS uses manual checks or Zod (covered in Module 08)
- The `loadConfig` wrapper pattern (`{ ok, config }` vs `{ ok, error }`) is a common JS pattern. It's similar to returning `Result` types — you'll see this formalized in Module 02 with TypeScript.
</details>

## Stretch Goal

Write a `loadConfigAll(jsonString)` that collects ALL validation errors instead of failing on the first one. On failure, return the errors as an array:

```javascript
loadConfigAll('{"model":"","batchSize":-1}');
// {
//   ok: false,
//   errors: [
//     "'model' is required and must be a non-empty string",
//     "'learningRate' is required and must be a positive number",
//     "'batchSize' is required and must be a positive integer",
//     "'epochs' is required and must be a positive integer",
//   ]
// }
```

<details>
<summary>Solution</summary>

```javascript
function loadConfigAll(jsonString) {
  let raw;
  try {
    raw = JSON.parse(jsonString);
  } catch (error) {
    return { ok: false, errors: [`Invalid JSON: ${error.message}`] };
  }

  const errors = [];

  if (typeof raw.model !== "string" || raw.model.length === 0) {
    errors.push("'model' is required and must be a non-empty string");
  }
  if (typeof raw.learningRate !== "number" || raw.learningRate <= 0) {
    errors.push("'learningRate' is required and must be a positive number");
  }
  if (!Number.isInteger(raw.batchSize) || raw.batchSize <= 0) {
    errors.push("'batchSize' is required and must be a positive integer");
  }
  if (!Number.isInteger(raw.epochs) || raw.epochs <= 0) {
    errors.push("'epochs' is required and must be a positive integer");
  }

  const validOptimizers = ["adam", "sgd", "adamw"];
  const optimizer = raw.optimizer ?? "adam";
  if (!validOptimizers.includes(optimizer)) {
    errors.push(`'optimizer' must be one of: ${validOptimizers.join(", ")}`);
  }

  if (errors.length > 0) {
    return { ok: false, errors };
  }

  return {
    ok: true,
    config: {
      model: raw.model,
      learningRate: raw.learningRate,
      batchSize: raw.batchSize,
      epochs: raw.epochs,
      optimizer,
      notes: raw.notes ?? "",
    },
  };
}
```
</details>
