# Exercise: Data Fetching

Add an API route that serves experiment data and fetch it from server and client components.

## Setup

Use your existing Next.js project from the previous exercises.

## Tasks

### 1. Create an API route

Create `app/api/experiments/route.ts`:

- `GET /api/experiments` — returns a JSON array of experiments
- Support a `?status=running` query parameter to filter by status
- Support a `?sort=name` or `?sort=accuracy` query parameter

This is similar to the Hono API routes from Module 08, but using Next.js's built-in API route system.

### 2. Fetch data in a server component

Update `app/experiments/page.tsx` to fetch data from the API route instead of importing hardcoded data:

- Use `fetch()` directly in the component (server components can be `async`)
- Display a loading state while data is being fetched (or handle it with Next.js's `loading.tsx`)

### 3. Add client-side interactivity

Create a client component that adds filtering and sorting to the experiments list:

- A search input to filter by name
- Dropdown to filter by status
- Click table headers to sort
- All filtering/sorting happens client-side on already-fetched data

### 4. Add a `loading.tsx` file

Create `app/experiments/loading.tsx` — Next.js automatically shows this while the page's server component is fetching data.

### 5. Add an error boundary

Create `app/experiments/error.tsx` — Next.js shows this if the server component throws an error.

## Key Concepts

**Server components** (the default in Next.js) can be `async` and fetch data directly:

```typescript
// This runs on the server — the browser never sees this fetch
export default async function Page() {
  const res = await fetch("http://localhost:3000/api/experiments");
  const data = await res.json();
  return <div>{/* render data */}</div>;
}
```

In Python terms, this is like a Jinja2 template where the view function fetches data and passes it to the template — except the "template" (React component) and the "view" (data fetching) are in the same function.

**Client components** handle interactivity. The pattern is: server component fetches data, passes it to a client component for interactive features.

```
ServerPage (fetches data)
  └── ClientTable (receives data as props, adds sorting/filtering)
```

<details>
<summary>Hint: API route structure</summary>

```typescript
// app/api/experiments/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const status = searchParams.get("status");
  // ... filter and return data
  return NextResponse.json(data);
}
```

This is similar to Hono's `c.req.query()` or FastAPI's `Query()` parameters.

</details>

<details>
<summary>Hint: loading.tsx and error.tsx</summary>

```typescript
// app/experiments/loading.tsx
export default function Loading() {
  return <p>Loading experiments...</p>;
}
```

```typescript
// app/experiments/error.tsx
"use client";  // error.tsx MUST be a client component

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

`loading.tsx` and `error.tsx` are Next.js conventions — they automatically wrap the page component in React Suspense and Error Boundary.

</details>

<details>
<summary>Solution</summary>

**`app/api/experiments/route.ts`**

```typescript
import { NextRequest, NextResponse } from "next/server";

interface Experiment {
  id: string;
  name: string;
  model: string;
  status: string;
  accuracy: number | null;
  createdAt: string;
}

const experiments: Experiment[] = [
  { id: "1", name: "BERT Sentiment", model: "bert-base", status: "completed", accuracy: 0.92, createdAt: "2024-01-15" },
  { id: "2", name: "GPT-2 Generation", model: "gpt2", status: "running", accuracy: null, createdAt: "2024-02-01" },
  { id: "3", name: "T5 Summary", model: "t5-small", status: "completed", accuracy: 0.87, createdAt: "2024-01-20" },
  { id: "4", name: "DistilBERT NER", model: "distilbert", status: "failed", accuracy: null, createdAt: "2024-02-10" },
  { id: "5", name: "RoBERTa QA", model: "roberta-base", status: "draft", accuracy: null, createdAt: "2024-02-15" },
  { id: "6", name: "ALBERT Classification", model: "albert-base", status: "completed", accuracy: 0.89, createdAt: "2024-01-25" },
];

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const status = searchParams.get("status");
  const sort = searchParams.get("sort");

  let result = [...experiments];

  // Filter by status
  if (status) {
    result = result.filter((e) => e.status === status);
  }

  // Sort
  if (sort === "name") {
    result.sort((a, b) => a.name.localeCompare(b.name));
  } else if (sort === "accuracy") {
    result.sort((a, b) => (b.accuracy ?? -1) - (a.accuracy ?? -1));
  }

  return NextResponse.json(result);
}
```

**`app/experiments/loading.tsx`**

```typescript
export default function Loading() {
  return (
    <div style={{ padding: "2rem" }}>
      <p>Loading experiments...</p>
    </div>
  );
}
```

**`app/experiments/error.tsx`**

```typescript
"use client";

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div style={{ padding: "2rem" }}>
      <h2>Something went wrong</h2>
      <p style={{ color: "red" }}>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

**`app/experiments/experiment-table.tsx`** — client component for interactivity

```typescript
"use client";

import { useState } from "react";
import StatusBadge from "../components/status-badge";

interface Experiment {
  id: string;
  name: string;
  model: string;
  status: string;
  accuracy: number | null;
  createdAt: string;
}

type SortField = "name" | "model" | "accuracy" | "createdAt";

export default function ExperimentTable({ experiments }: { experiments: Experiment[] }) {
  const [search, setSearch] = useState("");
  const [statusFilter, setStatusFilter] = useState("all");
  const [sortField, setSortField] = useState<SortField>("name");
  const [sortAsc, setSortAsc] = useState(true);

  // Filter
  let filtered = experiments.filter((e) =>
    e.name.toLowerCase().includes(search.toLowerCase())
  );
  if (statusFilter !== "all") {
    filtered = filtered.filter((e) => e.status === statusFilter);
  }

  // Sort
  filtered.sort((a, b) => {
    let cmp = 0;
    if (sortField === "accuracy") {
      cmp = (a.accuracy ?? -1) - (b.accuracy ?? -1);
    } else {
      const aVal = a[sortField] ?? "";
      const bVal = b[sortField] ?? "";
      cmp = String(aVal).localeCompare(String(bVal));
    }
    return sortAsc ? cmp : -cmp;
  });

  function handleSort(field: SortField) {
    if (sortField === field) {
      setSortAsc(!sortAsc);
    } else {
      setSortField(field);
      setSortAsc(true);
    }
  }

  const statuses = ["all", ...new Set(experiments.map((e) => e.status))];

  return (
    <div>
      {/* Controls */}
      <div style={{ display: "flex", gap: "1rem", marginBottom: "1rem" }}>
        <input
          type="text"
          placeholder="Search by name..."
          value={search}
          onChange={(e) => setSearch(e.target.value)}
          style={{ padding: "0.5rem", flex: 1 }}
        />
        <select
          value={statusFilter}
          onChange={(e) => setStatusFilter(e.target.value)}
          style={{ padding: "0.5rem" }}
        >
          {statuses.map((s) => (
            <option key={s} value={s}>
              {s === "all" ? "All statuses" : s}
            </option>
          ))}
        </select>
      </div>

      {/* Results count */}
      <p style={{ color: "#666", marginBottom: "0.5rem" }}>
        {filtered.length} experiment{filtered.length !== 1 ? "s" : ""}
      </p>

      {/* Table */}
      <table style={{ width: "100%", borderCollapse: "collapse" }}>
        <thead>
          <tr style={{ borderBottom: "2px solid #ccc", textAlign: "left" }}>
            <th
              style={{ padding: "0.5rem", cursor: "pointer" }}
              onClick={() => handleSort("name")}
            >
              Name {sortField === "name" ? (sortAsc ? "↑" : "↓") : ""}
            </th>
            <th
              style={{ padding: "0.5rem", cursor: "pointer" }}
              onClick={() => handleSort("model")}
            >
              Model {sortField === "model" ? (sortAsc ? "↑" : "↓") : ""}
            </th>
            <th style={{ padding: "0.5rem" }}>Status</th>
            <th
              style={{ padding: "0.5rem", cursor: "pointer" }}
              onClick={() => handleSort("accuracy")}
            >
              Accuracy {sortField === "accuracy" ? (sortAsc ? "↑" : "↓") : ""}
            </th>
          </tr>
        </thead>
        <tbody>
          {filtered.map((exp) => (
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

**`app/experiments/page.tsx`** — updated to fetch from API

```typescript
import ExperimentTable from "./experiment-table";

export default async function ExperimentsPage() {
  const res = await fetch("http://localhost:3000/api/experiments", {
    cache: "no-store",  // Always fetch fresh data
  });

  if (!res.ok) {
    throw new Error(`Failed to fetch experiments: ${res.status}`);
  }

  const experiments = await res.json();

  return (
    <div>
      <h1>Experiments</h1>
      <ExperimentTable experiments={experiments} />
    </div>
  );
}
```

### Testing

```bash
# Start the dev server
npm run dev

# Test the API directly
curl http://localhost:3000/api/experiments | jq
curl "http://localhost:3000/api/experiments?status=completed" | jq
curl "http://localhost:3000/api/experiments?sort=accuracy" | jq

# Visit the page
open http://localhost:3000/experiments
```

### Architecture diagram

```
Browser request: GET /experiments
        │
        ▼
┌─ Server ──────────────────────────────┐
│                                       │
│  ExperimentsPage (server component)   │
│    │                                  │
│    ├─ fetch("/api/experiments")        │
│    │     │                            │
│    │     └─► API Route (route.ts)     │
│    │           └─► returns JSON       │
│    │                                  │
│    └─ renders ExperimentTable         │
│         with data as props            │
│                                       │
└───────────────────────────────────────┘
        │
        ▼  (HTML + JS sent to browser)
┌─ Browser ─────────────────────────────┐
│                                       │
│  ExperimentTable (client component)   │
│    ├─ useState for search/filter/sort │
│    ├─ handles user interactions       │
│    └─ re-renders on state change      │
│                                       │
└───────────────────────────────────────┘
```

The server fetches data and renders the initial HTML. The client component hydrates in the browser and handles interactivity. The data fetch never happens in the browser — it's done server-side before the page is sent.

</details>
