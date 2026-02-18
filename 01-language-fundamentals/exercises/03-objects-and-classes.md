# Exercise: Objects and Classes

Build an ML experiment tracker using JavaScript classes.

## Goal

Create classes that model ML experiments, similar to what you'd build with Python dataclasses. This exercise practices class syntax, object manipulation, and array methods together.

## Requirements

### 1. Create a `Run` class

Each run represents a single training run of a model:

- Properties: `id` (string), `metrics` (object with `accuracy`, `loss`, `f1`), `status` (`"completed"` | `"failed"` | `"running"`)
- Constructor takes all properties

### 2. Create an `Experiment` class

An experiment groups multiple runs:

- Properties: `name` (string), `model` (string), `hyperparameters` (object), `runs` (array of Run)
- Constructor takes `name`, `model`, and `hyperparameters`. Runs start as empty array.
- Method: `addRun(run)` — adds a Run to the runs array
- Method: `completedRuns()` — returns only runs with status `"completed"`
- Method: `getBestRun()` — returns the completed run with highest accuracy, or `undefined` if no completed runs
- Method: `summarize()` — returns an object with `{ name, model, totalRuns, completedRuns, bestAccuracy }`

### 3. Test it

Create an experiment with a few runs and call each method:

```javascript
const exp = new Experiment("bert-finetune", "bert-base", {
  learningRate: 3e-5,
  batchSize: 32,
  epochs: 5,
});

exp.addRun(new Run("run-1", { accuracy: 0.89, loss: 0.34, f1: 0.87 }, "completed"));
exp.addRun(new Run("run-2", { accuracy: 0.92, loss: 0.28, f1: 0.91 }, "completed"));
exp.addRun(new Run("run-3", { accuracy: 0.0, loss: 5.2, f1: 0.0 }, "failed"));

console.log(exp.completedRuns().length);  // 2
console.log(exp.getBestRun().id);         // "run-2"
console.log(exp.summarize());
```

<details>
<summary>Hint: getBestRun</summary>

Use `.reduce()` on completed runs to find the one with highest accuracy:
```javascript
runs.reduce((best, run) => run.metrics.accuracy > best.metrics.accuracy ? run : best)
```
</details>

<details>
<summary>Solution</summary>

```javascript
class Run {
  constructor(id, metrics, status) {
    this.id = id;
    this.metrics = metrics;
    this.status = status;
  }
}

class Experiment {
  constructor(name, model, hyperparameters) {
    this.name = name;
    this.model = model;
    this.hyperparameters = hyperparameters;
    this.runs = [];
  }

  addRun(run) {
    this.runs.push(run);
  }

  completedRuns() {
    return this.runs.filter(r => r.status === "completed");
  }

  getBestRun() {
    const completed = this.completedRuns();
    if (completed.length === 0) return undefined;
    return completed.reduce((best, run) =>
      run.metrics.accuracy > best.metrics.accuracy ? run : best
    );
  }

  summarize() {
    const completed = this.completedRuns();
    const bestRun = this.getBestRun();
    return {
      name: this.name,
      model: this.model,
      totalRuns: this.runs.length,
      completedRuns: completed.length,
      bestAccuracy: bestRun ? bestRun.metrics.accuracy : null,
    };
  }
}

// Test it
const exp = new Experiment("bert-finetune", "bert-base", {
  learningRate: 3e-5,
  batchSize: 32,
  epochs: 5,
});

exp.addRun(new Run("run-1", { accuracy: 0.89, loss: 0.34, f1: 0.87 }, "completed"));
exp.addRun(new Run("run-2", { accuracy: 0.92, loss: 0.28, f1: 0.91 }, "completed"));
exp.addRun(new Run("run-3", { accuracy: 0.0, loss: 5.2, f1: 0.0 }, "failed"));

console.log("Completed:", exp.completedRuns().length);   // 2
console.log("Best run:", exp.getBestRun().id);            // "run-2"
console.log("Summary:", exp.summarize());
// { name: "bert-finetune", model: "bert-base", totalRuns: 3, completedRuns: 2, bestAccuracy: 0.92 }
```

### Python Equivalent for Comparison

```python
from dataclasses import dataclass, field

@dataclass
class Run:
    id: str
    metrics: dict[str, float]
    status: str

@dataclass
class Experiment:
    name: str
    model: str
    hyperparameters: dict
    runs: list[Run] = field(default_factory=list)

    def add_run(self, run: Run):
        self.runs.append(run)

    def completed_runs(self) -> list[Run]:
        return [r for r in self.runs if r.status == "completed"]

    def get_best_run(self) -> Run | None:
        completed = self.completed_runs()
        return max(completed, key=lambda r: r.metrics["accuracy"]) if completed else None

    def summarize(self) -> dict:
        best = self.get_best_run()
        return {
            "name": self.name,
            "model": self.model,
            "total_runs": len(self.runs),
            "completed_runs": len(self.completed_runs()),
            "best_accuracy": best.metrics["accuracy"] if best else None,
        }
```

Key differences:
- JS uses `constructor` instead of `@dataclass` / `__init__`
- JS assigns with `this.x = x` explicitly; Python dataclasses generate this
- JS uses `.filter()` where Python uses list comprehensions
- JS uses `.reduce()` where Python uses `max(key=...)`
- JS naming is camelCase; Python is snake_case
</details>

## Stretch Goal

Add a `static` method `Experiment.compare(experiments)` that takes an array of experiments and returns them ranked by best accuracy:

<details>
<summary>Solution</summary>

```javascript
class Experiment {
  // ... (previous code)

  static compare(experiments) {
    return [...experiments]
      .map(exp => ({
        name: exp.name,
        bestAccuracy: exp.getBestRun()?.metrics.accuracy ?? null,
      }))
      .sort((a, b) => (b.bestAccuracy ?? -1) - (a.bestAccuracy ?? -1));
  }
}
```

Note the `?.` optional chaining and `??` nullish coalescing — very handy for dealing with missing data.
</details>
