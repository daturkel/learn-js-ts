# Module 09: React and Next.js

Build web UIs with React components and full-stack apps with Next.js.

**Time:** 2-3 sessions

## Why React?

React is the dominant UI framework for web apps, especially AI tools. ChatGPT, Claude.ai, and most AI dashboards are React apps. Understanding React gives you access to the largest ecosystem of UI components and patterns.

## JSX — HTML in Your JavaScript

React uses JSX — a syntax that lets you write HTML-like markup inside JavaScript:

```typescript
// This is JSX — it looks like HTML but it's JavaScript
const element = <h1>Hello, World!</h1>;

// With dynamic content (like Python f-strings, but with {})
const name = "Alice";
const greeting = <h1>Hello, {name}!</h1>;

// With expressions
const items = ["PyTorch", "TensorFlow", "JAX"];
const list = (
  <ul>
    {items.map(item => <li key={item}>{item}</li>)}
  </ul>
);
```

JSX differences from HTML:
- `className` instead of `class` (because `class` is a JS keyword)
- `onClick` instead of `onclick` (camelCase)
- Self-closing tags required: `<img />` not `<img>`
- Everything must have one root element (or use `<>...</>` fragments)

If you've used Jinja2 templates, JSX is a similar idea but more powerful — the "template" is actual code with full access to JavaScript.

## Components

Components are functions that return JSX. They're the building blocks of React UIs.

```typescript
// A simple component
function Welcome() {
  return <h1>Welcome to the Dashboard</h1>;
}

// Use it like an HTML tag
function App() {
  return (
    <div>
      <Welcome />
      <p>Content here</p>
    </div>
  );
}
```

Think of components as Python functions that return HTML instead of data.

## Props — Passing Data to Components

Props are like function arguments:

```typescript
// Define props with a TypeScript interface
interface UserCardProps {
  name: string;
  email: string;
  role?: string;  // Optional prop
}

function UserCard({ name, email, role = "user" }: UserCardProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      <span>{role}</span>
    </div>
  );
}

// Use it
function App() {
  return (
    <div>
      <UserCard name="Alice" email="alice@example.com" role="admin" />
      <UserCard name="Bob" email="bob@example.com" />
    </div>
  );
}
```

## State — Interactive Data

`useState` creates reactive data — when it changes, the component re-renders:

```typescript
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);
  //     ^value  ^setter      ^initial value

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

State is like an instance variable that automatically re-renders the UI when changed. In Python terms: imagine a variable that calls a `render()` function every time you assign to it.

**Important:** Never mutate state directly. Always use the setter:

```typescript
// WRONG — mutating doesn't trigger re-render
items.push(newItem);

// RIGHT — create new array
setItems([...items, newItem]);
```

## Hooks

React hooks are functions that let components use features like state and side effects.

### useEffect — Side Effects

Runs code when the component mounts or when dependencies change:

```typescript
import { useState, useEffect } from "react";

function ExperimentList() {
  const [experiments, setExperiments] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // This runs after the component renders
    fetch("/api/experiments")
      .then(res => res.json())
      .then(data => {
        setExperiments(data);
        setLoading(false);
      });
  }, []);  // Empty array = run once on mount

  if (loading) return <p>Loading...</p>;

  return (
    <ul>
      {experiments.map(exp => <li key={exp.id}>{exp.name}</li>)}
    </ul>
  );
}
```

The dependency array controls when the effect re-runs:
- `[]` — run once on mount (like `__init__`)
- `[query]` — re-run when `query` changes
- No array — re-run on every render (usually a mistake)

### useRef — Mutable Value Without Re-renders

```typescript
import { useRef } from "react";

function Timer() {
  const intervalRef = useRef<number | null>(null);

  function start() {
    intervalRef.current = setInterval(() => console.log("tick"), 1000);
  }

  function stop() {
    if (intervalRef.current) clearInterval(intervalRef.current);
  }

  return (
    <div>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

## Event Handling

```typescript
function Form() {
  const [value, setValue] = useState("");

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();  // Prevent page reload
    console.log("Submitted:", value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Enter text"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Next.js — Full-Stack React

Next.js is a framework built on React that adds routing, server-side rendering, and API routes. Think of it as "FastAPI + Jinja2 in one, but templates are React components."

### Creating a Next.js App

```bash
npx create-next-app@latest my-app --typescript --app --tailwind
cd my-app
npm run dev    # Start dev server on http://localhost:3000
```

### File-Based Routing

Files in `app/` automatically become routes:

```
app/
├── page.tsx              → /
├── about/
│   └── page.tsx          → /about
├── experiments/
│   ├── page.tsx          → /experiments
│   └── [id]/
│       └── page.tsx      → /experiments/:id
└── layout.tsx            → Shared layout for all pages
```

### Pages

```typescript
// app/page.tsx — the home page
export default function Home() {
  return (
    <main>
      <h1>ML Dashboard</h1>
      <p>Welcome to the experiment tracker.</p>
    </main>
  );
}
```

### Layouts — Shared UI

```typescript
// app/layout.tsx — wraps all pages
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <nav>
          <a href="/">Home</a>
          <a href="/experiments">Experiments</a>
        </nav>
        <main>{children}</main>
      </body>
    </html>
  );
}
```

### Server vs Client Components

This is the biggest conceptual shift in modern React:

**Server Components** (default) — run on the server:
- Can be `async` and fetch data directly
- No interactivity (no useState, onClick, etc.)
- Don't ship JavaScript to the browser

**Client Components** — run in the browser:
- Add `"use client"` at the top of the file
- Can use hooks (useState, useEffect)
- Handle user interactions

```typescript
// Server component (default) — fetches data on the server
// app/experiments/page.tsx
async function ExperimentsPage() {
  const res = await fetch("http://localhost:3000/api/experiments");
  const experiments = await res.json();

  return (
    <div>
      <h1>Experiments</h1>
      <ExperimentList experiments={experiments} />
    </div>
  );
}

// Client component — handles interactivity
// app/experiments/experiment-list.tsx
"use client";
import { useState } from "react";

function ExperimentList({ experiments }) {
  const [search, setSearch] = useState("");
  const filtered = experiments.filter(e =>
    e.name.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div>
      <input value={search} onChange={e => setSearch(e.target.value)} placeholder="Search..." />
      <ul>
        {filtered.map(exp => <li key={exp.id}>{exp.name}</li>)}
      </ul>
    </div>
  );
}
```

### API Routes

```typescript
// app/api/experiments/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  const experiments = [/* data */];
  return NextResponse.json(experiments);
}

export async function POST(request: Request) {
  const body = await request.json();
  return NextResponse.json({ created: body }, { status: 201 });
}
```

## Further Reading

- [React Official Tutorial](https://react.dev/learn) — Start here for React fundamentals
- [Next.js Learn Course](https://nextjs.org/learn) — Hands-on Next.js tutorial
- [Next.js App Router Docs](https://nextjs.org/docs/app) — Reference documentation

## Exercises

1. [First React App](./exercises/01-first-react-app.md) — Create and explore a Next.js project
2. [Components and State](./exercises/02-components-and-state.md) — Build an interactive form
3. [Next.js App](./exercises/03-nextjs-app.md) — Multi-page experiment dashboard
4. [Data Fetching](./exercises/04-data-fetching.md) — API routes and server components

---

**Next:** [Exercise 1: First React App →](/09-react-and-nextjs/exercises/01-first-react-app.md)
