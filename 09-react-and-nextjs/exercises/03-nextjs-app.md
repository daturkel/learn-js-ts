# Exercise: Multi-Page Next.js App

Build a multi-page experiment dashboard with file-based routing, layouts, and dynamic routes.

## Setup

Use your existing Next.js project or create a new one.

## Tasks

### 1. Create the experiments list page

Create `app/experiments/page.tsx`:

- Display a list of experiments with name, model, and status
- Each experiment should link to its detail page
- Use hardcoded data for now (no API calls yet — that's Exercise 04)

Sample data:

```typescript
const experiments = [
  { id: "1", name: "BERT Sentiment", model: "bert-base", status: "completed", accuracy: 0.92 },
  { id: "2", name: "GPT-2 Generation", model: "gpt2", status: "running", accuracy: null },
  { id: "3", name: "T5 Summary", model: "t5-small", status: "completed", accuracy: 0.87 },
  { id: "4", name: "DistilBERT NER", model: "distilbert", status: "failed", accuracy: null },
  { id: "5", name: "RoBERTa QA", model: "roberta-base", status: "draft", accuracy: null },
];
```

### 2. Create a dynamic experiment detail page

Create `app/experiments/[id]/page.tsx`:

- The `[id]` folder makes this a dynamic route — `/experiments/1`, `/experiments/2`, etc.
- Display all details for the experiment matching the `id` param
- Show a "Back to experiments" link
- Show "Experiment not found" for invalid IDs

### 3. Add an experiments layout

Create `app/experiments/layout.tsx`:

- Add a sidebar or header specific to the experiments section
- This layout wraps all pages under `/experiments/*` but NOT the home page or about page

### 4. Style the status badges

Create a `StatusBadge` component that color-codes the experiment status:
- `completed` → green
- `running` → blue
- `failed` → red
- `draft` → gray

### 5. Update navigation

Add an "Experiments" link to the root layout's nav bar.

## Verify

Run `npm run dev` and check:
- `http://localhost:3000/experiments` — table of 5 experiments, each linking to its detail page
- `http://localhost:3000/experiments/1` — detail page for BERT Sentiment
- `http://localhost:3000/experiments/999` — "Experiment not found" message
- Any `/experiments/*` page — sidebar appears; home page — sidebar does NOT appear
- Status badges are color-coded (green/blue/red/gray)
- Nav bar includes an "Experiments" link

## Key Concepts

**Dynamic routes** use bracket folders: `[id]`, `[slug]`, etc. The param is available in the page component's props:

```typescript
// app/experiments/[id]/page.tsx
export default function ExperimentPage({ params }: { params: { id: string } }) {
  // params.id is the value from the URL
}
```

This is similar to FastAPI's path parameters:
```python
@app.get("/experiments/{id}")
def get_experiment(id: str):
    ...
```

**Nested layouts** compose automatically. If you have:
```
app/layout.tsx              ← wraps everything
app/experiments/layout.tsx  ← wraps only /experiments/*
```

Then `/experiments/3` gets both layouts: root layout (nav bar) wrapping experiments layout (sidebar) wrapping the page content.

<details>
<summary>Hint: Reading params in a page component</summary>

In the Next.js App Router, page components receive `params` as a prop. With recent versions of Next.js, `params` is a Promise that you need to `await`:

```typescript
export default async function ExperimentPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  // use id...
}
```

If you're on an older version, `params` might be a plain object. Check your Next.js version and the console for warnings.

</details>

<details>
<summary>Solution</summary>

**`app/components/status-badge.tsx`**

```typescript
const STATUS_COLORS: Record<string, { bg: string; text: string }> = {
  completed: { bg: "#dcfce7", text: "#166534" },
  running:   { bg: "#dbeafe", text: "#1e40af" },
  failed:    { bg: "#fee2e2", text: "#991b1b" },
  draft:     { bg: "#f3f4f6", text: "#374151" },
};

export default function StatusBadge({ status }: { status: string }) {
  const colors = STATUS_COLORS[status] ?? STATUS_COLORS.draft;

  return (
    <span
      style={{
        padding: "0.25rem 0.75rem",
        borderRadius: "9999px",
        fontSize: "0.875rem",
        fontWeight: 500,
        backgroundColor: colors.bg,
        color: colors.text,
      }}
    >
      {status}
    </span>
  );
}
```

**`app/experiments/data.ts`** — shared data (imported by both pages)

```typescript
export interface Experiment {
  id: string;
  name: string;
  model: string;
  status: string;
  accuracy: number | null;
}

export const experiments: Experiment[] = [
  { id: "1", name: "BERT Sentiment", model: "bert-base", status: "completed", accuracy: 0.92 },
  { id: "2", name: "GPT-2 Generation", model: "gpt2", status: "running", accuracy: null },
  { id: "3", name: "T5 Summary", model: "t5-small", status: "completed", accuracy: 0.87 },
  { id: "4", name: "DistilBERT NER", model: "distilbert", status: "failed", accuracy: null },
  { id: "5", name: "RoBERTa QA", model: "roberta-base", status: "draft", accuracy: null },
];
```

**`app/experiments/layout.tsx`**

```typescript
export default function ExperimentsLayout({ children }: { children: React.ReactNode }) {
  return (
    <div style={{ display: "flex" }}>
      <aside style={{
        width: "200px",
        padding: "1rem",
        borderRight: "1px solid #ccc",
        minHeight: "calc(100vh - 60px)",
      }}>
        <h3>Experiments</h3>
        <ul style={{ listStyle: "none", padding: 0 }}>
          <li><a href="/experiments">All Experiments</a></li>
        </ul>
      </aside>
      <div style={{ flex: 1, padding: "1rem" }}>
        {children}
      </div>
    </div>
  );
}
```

**`app/experiments/page.tsx`**

```typescript
import { experiments } from "./data";
import StatusBadge from "../components/status-badge";

export default function ExperimentsPage() {
  return (
    <div>
      <h1>Experiments</h1>
      <table style={{ width: "100%", borderCollapse: "collapse" }}>
        <thead>
          <tr style={{ borderBottom: "2px solid #ccc", textAlign: "left" }}>
            <th style={{ padding: "0.5rem" }}>Name</th>
            <th style={{ padding: "0.5rem" }}>Model</th>
            <th style={{ padding: "0.5rem" }}>Status</th>
            <th style={{ padding: "0.5rem" }}>Accuracy</th>
          </tr>
        </thead>
        <tbody>
          {experiments.map((exp) => (
            <tr key={exp.id} style={{ borderBottom: "1px solid #eee" }}>
              <td style={{ padding: "0.5rem" }}>
                <a href={`/experiments/${exp.id}`}>{exp.name}</a>
              </td>
              <td style={{ padding: "0.5rem" }}>{exp.model}</td>
              <td style={{ padding: "0.5rem" }}>
                <StatusBadge status={exp.status} />
              </td>
              <td style={{ padding: "0.5rem" }}>
                {exp.accuracy !== null ? `${(exp.accuracy * 100).toFixed(1)}%` : "—"}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

**`app/experiments/[id]/page.tsx`**

```typescript
import { experiments } from "../data";
import StatusBadge from "../../components/status-badge";

export default async function ExperimentDetailPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const experiment = experiments.find((e) => e.id === id);

  if (!experiment) {
    return (
      <div>
        <h1>Experiment Not Found</h1>
        <p>No experiment with ID "{id}".</p>
        <a href="/experiments">← Back to experiments</a>
      </div>
    );
  }

  return (
    <div>
      <a href="/experiments">← Back to experiments</a>
      <h1>{experiment.name}</h1>
      <StatusBadge status={experiment.status} />
      <table style={{ marginTop: "1rem" }}>
        <tbody>
          <tr>
            <td><strong>ID</strong></td>
            <td>{experiment.id}</td>
          </tr>
          <tr>
            <td><strong>Model</strong></td>
            <td>{experiment.model}</td>
          </tr>
          <tr>
            <td><strong>Status</strong></td>
            <td>{experiment.status}</td>
          </tr>
          <tr>
            <td><strong>Accuracy</strong></td>
            <td>
              {experiment.accuracy !== null
                ? `${(experiment.accuracy * 100).toFixed(1)}%`
                : "Not available"}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  );
}
```

**Update `app/layout.tsx`** — add experiments link:

```typescript
<nav style={{ padding: "1rem", borderBottom: "1px solid #ccc", display: "flex", gap: "1rem" }}>
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/tuning">Tuning</a>
  <a href="/experiments">Experiments</a>
</nav>
```

### File structure

```
app/
├── layout.tsx                    ← root layout (nav bar)
├── page.tsx                      ← home page
├── about/
│   └── page.tsx                  ← /about
├── tuning/
│   └── page.tsx                  ← /tuning (from Exercise 02)
├── components/
│   ├── hyperparameter-slider.tsx
│   ├── model-selector.tsx
│   ├── results-display.tsx
│   └── status-badge.tsx
└── experiments/
    ├── data.ts                   ← shared experiment data
    ├── layout.tsx                ← experiments layout (sidebar)
    ├── page.tsx                  ← /experiments (list)
    └── [id]/
        └── page.tsx              ← /experiments/:id (detail)
```

### How routing works

The file structure directly maps to URLs. No router configuration file needed — this is what Next.js calls "file-based routing." Compare to Python/FastAPI where you'd define each route with a decorator:

```python
# FastAPI — explicit route definitions
@app.get("/experiments")
@app.get("/experiments/{id}")
```

In Next.js, the file location IS the route definition.

</details>

---

**Next:** [Exercise 4: Data Fetching →](/09-react-and-nextjs/exercises/04-data-fetching.md)
