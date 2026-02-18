# Exercise: Type a Python Script

Rewrite a complete Python script in TypeScript, mapping Python types to TS types.

## The Python Script

Here's a data processing script that analyzes ML experiment results. Your job is to rewrite it in TypeScript.

```python
from dataclasses import dataclass
from typing import Optional
from datetime import datetime


@dataclass
class ExperimentResult:
    name: str
    model: str
    accuracy: float
    f1_score: float
    training_time_hours: float
    status: str  # "success" | "failed" | "timeout"
    error_message: Optional[str] = None


def load_results() -> list[ExperimentResult]:
    """Simulates loading experiment results from a file."""
    return [
        ExperimentResult("bert-v1", "bert", 0.912, 0.905, 2.5, "success"),
        ExperimentResult("bert-v2", "bert", 0.934, 0.928, 4.0, "success"),
        ExperimentResult("gpt2-v1", "gpt2", 0.887, 0.881, 1.5, "success"),
        ExperimentResult("gpt2-v2", "gpt2", 0.0, 0.0, 0.5, "failed", "OOM error"),
        ExperimentResult("t5-v1", "t5", 0.921, 0.915, 3.0, "success"),
        ExperimentResult("t5-v2", "t5", 0.0, 0.0, 6.0, "timeout"),
        ExperimentResult("roberta-v1", "roberta", 0.945, 0.940, 5.0, "success"),
    ]


def filter_successful(results: list[ExperimentResult]) -> list[ExperimentResult]:
    return [r for r in results if r.status == "success"]


def best_per_model(results: list[ExperimentResult]) -> dict[str, ExperimentResult]:
    best: dict[str, ExperimentResult] = {}
    for r in results:
        if r.model not in best or r.accuracy > best[r.model].accuracy:
            best[r.model] = r
    return best


def compute_summary(results: list[ExperimentResult]) -> dict:
    successful = filter_successful(results)
    failed = [r for r in results if r.status != "success"]

    avg_accuracy = sum(r.accuracy for r in successful) / len(successful) if successful else 0
    avg_training_time = sum(r.training_time_hours for r in successful) / len(successful) if successful else 0
    best = best_per_model(successful)

    return {
        "total_experiments": len(results),
        "successful": len(successful),
        "failed": len(failed),
        "average_accuracy": round(avg_accuracy, 4),
        "average_training_hours": round(avg_training_time, 2),
        "best_per_model": {
            model: {"name": r.name, "accuracy": r.accuracy}
            for model, r in best.items()
        },
        "failure_reasons": [
            {"name": r.name, "status": r.status, "error": r.error_message}
            for r in failed
        ],
    }


def main():
    results = load_results()
    summary = compute_summary(results)

    print(f"Total experiments: {summary['total_experiments']}")
    print(f"Success rate: {summary['successful']}/{summary['total_experiments']}")
    print(f"Average accuracy: {summary['average_accuracy']}")
    print(f"Average training time: {summary['average_training_hours']}h")
    print("\nBest per model:")
    for model, info in summary["best_per_model"].items():
        print(f"  {model}: {info['name']} ({info['accuracy']:.3f})")
    print("\nFailures:")
    for f in summary["failure_reasons"]:
        print(f"  {f['name']}: {f['status']} - {f['error'] or 'no details'}")


if __name__ == "__main__":
    main()
```

## Your Task

1. Rewrite this entire script in TypeScript (`experiment-analysis.ts`)
2. Define proper interfaces for all data structures (not just `dict`)
3. Use TypeScript's type system to make the code safer than the Python version
4. Run with `npx tsx experiment-analysis.ts` — the output should match

### What to Focus On

- `@dataclass` → TypeScript `interface` (or `class` — interface is more common in TS)
- `Optional[str]` → `string | undefined` (or `string?` in interface properties)
- `list[X]` → `X[]`
- `dict[str, X]` → `Record<string, X>` (or a typed interface)
- `status: str` → use a union type for the valid values
- The return type of `compute_summary` is `dict` — define a proper interface

<details>
<summary>Hint: The status type</summary>

```typescript
type Status = "success" | "failed" | "timeout";
```

This is much safer than Python's plain `str`. TypeScript will catch typos like `"sucess"` at compile time.
</details>

<details>
<summary>Hint: Rounding numbers</summary>

JavaScript has no built-in `round(x, n)`. Use:
```typescript
Math.round(x * 10000) / 10000  // 4 decimal places
// or
parseFloat(x.toFixed(4))
```
</details>

<details>
<summary>Solution</summary>

```typescript
// --- Type Definitions ---

type Status = "success" | "failed" | "timeout";

interface ExperimentResult {
  name: string;
  model: string;
  accuracy: number;
  f1Score: number;
  trainingTimeHours: number;
  status: Status;
  errorMessage?: string;
}

interface ModelBest {
  name: string;
  accuracy: number;
}

interface FailureInfo {
  name: string;
  status: Status;
  error: string | undefined;
}

interface Summary {
  totalExperiments: number;
  successful: number;
  failed: number;
  averageAccuracy: number;
  averageTrainingHours: number;
  bestPerModel: Record<string, ModelBest>;
  failureReasons: FailureInfo[];
}

// --- Functions ---

function loadResults(): ExperimentResult[] {
  return [
    { name: "bert-v1", model: "bert", accuracy: 0.912, f1Score: 0.905, trainingTimeHours: 2.5, status: "success" },
    { name: "bert-v2", model: "bert", accuracy: 0.934, f1Score: 0.928, trainingTimeHours: 4.0, status: "success" },
    { name: "gpt2-v1", model: "gpt2", accuracy: 0.887, f1Score: 0.881, trainingTimeHours: 1.5, status: "success" },
    { name: "gpt2-v2", model: "gpt2", accuracy: 0.0, f1Score: 0.0, trainingTimeHours: 0.5, status: "failed", errorMessage: "OOM error" },
    { name: "t5-v1", model: "t5", accuracy: 0.921, f1Score: 0.915, trainingTimeHours: 3.0, status: "success" },
    { name: "t5-v2", model: "t5", accuracy: 0.0, f1Score: 0.0, trainingTimeHours: 6.0, status: "timeout" },
    { name: "roberta-v1", model: "roberta", accuracy: 0.945, f1Score: 0.940, trainingTimeHours: 5.0, status: "success" },
  ];
}

function filterSuccessful(results: ExperimentResult[]): ExperimentResult[] {
  return results.filter(r => r.status === "success");
}

function bestPerModel(results: ExperimentResult[]): Record<string, ExperimentResult> {
  const best: Record<string, ExperimentResult> = {};
  for (const r of results) {
    if (!(r.model in best) || r.accuracy > best[r.model].accuracy) {
      best[r.model] = r;
    }
  }
  return best;
}

function computeSummary(results: ExperimentResult[]): Summary {
  const successful = filterSuccessful(results);
  const failed = results.filter(r => r.status !== "success");

  const avgAccuracy = successful.length > 0
    ? successful.reduce((sum, r) => sum + r.accuracy, 0) / successful.length
    : 0;

  const avgTrainingTime = successful.length > 0
    ? successful.reduce((sum, r) => sum + r.trainingTimeHours, 0) / successful.length
    : 0;

  const best = bestPerModel(successful);

  const bestPerModelSummary: Record<string, ModelBest> = {};
  for (const [model, r] of Object.entries(best)) {
    bestPerModelSummary[model] = { name: r.name, accuracy: r.accuracy };
  }

  return {
    totalExperiments: results.length,
    successful: successful.length,
    failed: failed.length,
    averageAccuracy: parseFloat(avgAccuracy.toFixed(4)),
    averageTrainingHours: parseFloat(avgTrainingTime.toFixed(2)),
    bestPerModel: bestPerModelSummary,
    failureReasons: failed.map(r => ({
      name: r.name,
      status: r.status,
      error: r.errorMessage,
    })),
  };
}

function main() {
  const results = loadResults();
  const summary = computeSummary(results);

  console.log(`Total experiments: ${summary.totalExperiments}`);
  console.log(`Success rate: ${summary.successful}/${summary.totalExperiments}`);
  console.log(`Average accuracy: ${summary.averageAccuracy}`);
  console.log(`Average training time: ${summary.averageTrainingHours}h`);
  console.log("\nBest per model:");
  for (const [model, info] of Object.entries(summary.bestPerModel)) {
    console.log(`  ${model}: ${info.name} (${info.accuracy.toFixed(3)})`);
  }
  console.log("\nFailures:");
  for (const f of summary.failureReasons) {
    console.log(`  ${f.name}: ${f.status} - ${f.error ?? "no details"}`);
  }
}

main();
```

### Key Observations

1. **Union type for Status** — Python used a plain `str`, allowing any value. TS restricts to valid values at compile time.
2. **Interfaces for everything** — Python's `compute_summary` returned `dict` (untyped). TS forces you to define `Summary`, making the return shape explicit.
3. **No dataclass magic** — TS interfaces have zero runtime cost. Python `@dataclass` generates `__init__`, `__repr__`, etc. In TS, you just use plain objects that match the interface shape.
4. **`snake_case` → `camelCase`** — JS convention. Your IDE will remind you.
5. **`??` instead of `or`** — Python uses `f.error or "no details"`. TS uses `f.error ?? "no details"`.
6. **`for...of` instead of `for...in`** — Python's `for x in list` is JS's `for (const x of list)`.
</details>
