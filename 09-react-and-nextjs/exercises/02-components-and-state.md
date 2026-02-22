# Exercise: Components and State

Build an interactive ML hyperparameter tuning form using React components and `useState`.

## Setup

Use the Next.js project from Exercise 01 (or create a new one).

## Tasks

### 1. Create a `HyperparameterSlider` component

Create `app/components/hyperparameter-slider.tsx`:

```typescript
"use client";

interface HyperparameterSliderProps {
  label: string;
  value: number;
  min: number;
  max: number;
  step: number;
  onChange: (value: number) => void;
}
```

The component should:
- Display the label and current value
- Render an `<input type="range">` slider
- Call `onChange` when the slider moves

### 2. Create a `ModelSelector` component

Create `app/components/model-selector.tsx`:

```typescript
"use client";

interface ModelSelectorProps {
  models: string[];
  selected: string;
  onSelect: (model: string) => void;
}
```

The component should:
- Display radio buttons or a `<select>` dropdown for each model
- Highlight the currently selected model
- Call `onSelect` when a new model is chosen

### 3. Create a `ResultsDisplay` component

Create `app/components/results-display.tsx`:

```typescript
interface ResultsDisplayProps {
  model: string;
  hyperparameters: Record<string, number>;
}
```

The component should:
- Display the current model and all hyperparameters in a formatted view
- Show a fake "estimated accuracy" calculated from the hyperparameters (just make up a formula)
- This component does NOT need `"use client"` — it receives data via props and has no interactivity

### 4. Build the tuning page

Create `app/tuning/page.tsx` that combines everything:

- A `ModelSelector` with options like `["bert-base", "gpt2", "t5-small", "distilbert"]`
- Three `HyperparameterSlider` components:
  - Learning rate: 0.0001 to 0.01, step 0.0001
  - Epochs: 1 to 20, step 1
  - Batch size: 8 to 128, step 8
- A `ResultsDisplay` showing the current configuration
- A "Reset Defaults" button

All state lives in the page component and is passed down via props.

### 5. Add navigation

Update `app/layout.tsx` to include a link to `/tuning`.

## Key Concepts

**State lives in the parent, not the child.** The page owns the state. Children receive values via props and notify the parent via callback functions (`onChange`, `onSelect`). This is called "lifting state up."

In Python terms:
```python
# React's pattern is similar to this
class Parent:
    def __init__(self):
        self.value = 0

    def handle_change(self, new_value):
        self.value = new_value
        self.render()

    def render(self):
        # Pass value and callback to child
        child = Slider(value=self.value, on_change=self.handle_change)
```

**`"use client"` is needed** for any component that uses `useState`, `useEffect`, event handlers, or browser APIs. Components that just render props don't need it.

<details>
<summary>Hint: Slider component</summary>

```typescript
"use client";

interface HyperparameterSliderProps {
  label: string;
  value: number;
  min: number;
  max: number;
  step: number;
  onChange: (value: number) => void;
}

export default function HyperparameterSlider({
  label, value, min, max, step, onChange
}: HyperparameterSliderProps) {
  return (
    <div style={{ marginBottom: "1rem" }}>
      <label>
        {label}: <strong>{value}</strong>
      </label>
      <br />
      <input
        type="range"
        min={min}
        max={max}
        step={step}
        value={value}
        onChange={(e) => onChange(parseFloat(e.target.value))}
        style={{ width: "300px" }}
      />
    </div>
  );
}
```

Note: `e.target.value` is always a string, even for number inputs. You must convert it with `parseFloat()` or `Number()`.

</details>

<details>
<summary>Solution</summary>

**`app/components/hyperparameter-slider.tsx`**

```typescript
"use client";

interface HyperparameterSliderProps {
  label: string;
  value: number;
  min: number;
  max: number;
  step: number;
  onChange: (value: number) => void;
}

export default function HyperparameterSlider({
  label, value, min, max, step, onChange
}: HyperparameterSliderProps) {
  return (
    <div style={{ marginBottom: "1rem" }}>
      <label>
        {label}: <strong>{value}</strong>
      </label>
      <br />
      <input
        type="range"
        min={min}
        max={max}
        step={step}
        value={value}
        onChange={(e) => onChange(parseFloat(e.target.value))}
        style={{ width: "300px" }}
      />
    </div>
  );
}
```

**`app/components/model-selector.tsx`**

```typescript
"use client";

interface ModelSelectorProps {
  models: string[];
  selected: string;
  onSelect: (model: string) => void;
}

export default function ModelSelector({ models, selected, onSelect }: ModelSelectorProps) {
  return (
    <div style={{ marginBottom: "1.5rem" }}>
      <h3>Model</h3>
      <div style={{ display: "flex", gap: "1rem", flexWrap: "wrap" }}>
        {models.map((model) => (
          <label key={model} style={{ cursor: "pointer" }}>
            <input
              type="radio"
              name="model"
              value={model}
              checked={model === selected}
              onChange={() => onSelect(model)}
            />
            {" "}{model}
          </label>
        ))}
      </div>
    </div>
  );
}
```

**`app/components/results-display.tsx`**

```typescript
interface ResultsDisplayProps {
  model: string;
  hyperparameters: Record<string, number>;
}

export default function ResultsDisplay({ model, hyperparameters }: ResultsDisplayProps) {
  // Fake accuracy formula — just for demonstration
  const lr = hyperparameters["learningRate"] ?? 0.001;
  const epochs = hyperparameters["epochs"] ?? 3;
  const batchSize = hyperparameters["batchSize"] ?? 32;

  const baseAccuracy = model.includes("bert") ? 0.88 : model.includes("gpt") ? 0.85 : 0.82;
  const lrBonus = lr > 0.005 ? -0.05 : lr < 0.0005 ? -0.03 : 0.02;
  const epochBonus = Math.min(epochs * 0.008, 0.1);
  const batchPenalty = batchSize > 64 ? -0.02 : 0;
  const accuracy = Math.min(baseAccuracy + lrBonus + epochBonus + batchPenalty, 0.99);

  return (
    <div style={{ padding: "1rem", border: "1px solid #ccc", borderRadius: "8px" }}>
      <h3>Configuration Summary</h3>
      <table>
        <tbody>
          <tr><td><strong>Model</strong></td><td>{model}</td></tr>
          {Object.entries(hyperparameters).map(([key, value]) => (
            <tr key={key}>
              <td><strong>{key}</strong></td>
              <td>{value}</td>
            </tr>
          ))}
        </tbody>
      </table>
      <p style={{ marginTop: "1rem", fontSize: "1.2rem" }}>
        Estimated accuracy: <strong>{(accuracy * 100).toFixed(1)}%</strong>
      </p>
    </div>
  );
}
```

**`app/tuning/page.tsx`**

```typescript
"use client";

import { useState } from "react";
import HyperparameterSlider from "../components/hyperparameter-slider";
import ModelSelector from "../components/model-selector";
import ResultsDisplay from "../components/results-display";

const MODELS = ["bert-base", "gpt2", "t5-small", "distilbert"];

const DEFAULTS = {
  model: "bert-base",
  learningRate: 0.001,
  epochs: 3,
  batchSize: 32,
};

export default function TuningPage() {
  const [model, setModel] = useState(DEFAULTS.model);
  const [learningRate, setLearningRate] = useState(DEFAULTS.learningRate);
  const [epochs, setEpochs] = useState(DEFAULTS.epochs);
  const [batchSize, setBatchSize] = useState(DEFAULTS.batchSize);

  function resetDefaults() {
    setModel(DEFAULTS.model);
    setLearningRate(DEFAULTS.learningRate);
    setEpochs(DEFAULTS.epochs);
    setBatchSize(DEFAULTS.batchSize);
  }

  return (
    <main style={{ padding: "2rem", maxWidth: "600px" }}>
      <h1>Hyperparameter Tuning</h1>

      <ModelSelector models={MODELS} selected={model} onSelect={setModel} />

      <h3>Hyperparameters</h3>
      <HyperparameterSlider
        label="Learning Rate"
        value={learningRate}
        min={0.0001}
        max={0.01}
        step={0.0001}
        onChange={setLearningRate}
      />
      <HyperparameterSlider
        label="Epochs"
        value={epochs}
        min={1}
        max={20}
        step={1}
        onChange={setEpochs}
      />
      <HyperparameterSlider
        label="Batch Size"
        value={batchSize}
        min={8}
        max={128}
        step={8}
        onChange={setBatchSize}
      />

      <button onClick={resetDefaults} style={{ marginBottom: "1.5rem" }}>
        Reset Defaults
      </button>

      <ResultsDisplay
        model={model}
        hyperparameters={{ learningRate, epochs, batchSize }}
      />
    </main>
  );
}
```

**Update `app/layout.tsx`** — add the tuning link:

```typescript
<nav style={{ padding: "1rem", borderBottom: "1px solid #ccc", display: "flex", gap: "1rem" }}>
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/tuning">Tuning</a>
</nav>
```

### How data flows

```
TuningPage (owns all state)
├── ModelSelector      ← receives: models, selected, onSelect
├── HyperparameterSlider (×3) ← receives: label, value, min/max/step, onChange
└── ResultsDisplay     ← receives: model, hyperparameters
```

Every component is a pure function of its props. State changes flow: user interaction → callback → setState → re-render → new props → children update.

</details>
