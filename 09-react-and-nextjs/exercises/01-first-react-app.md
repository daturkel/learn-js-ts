# Exercise: First React App

Create a Next.js project and explore the file structure.

## Steps

### 1. Create the project

```bash
npx create-next-app@latest my-dashboard --typescript --app --tailwind --eslint --no-src-dir --import-alias "@/*"
cd my-dashboard
npm run dev
```

Open http://localhost:3000 — you should see the Next.js starter page.

### 2. Explore the file structure

Look at each file and understand its role:

- `app/layout.tsx` — root layout wrapping all pages
- `app/page.tsx` — the home page (`/`)
- `app/globals.css` — global styles
- `next.config.ts` — Next.js configuration
- `tsconfig.json` — TypeScript config
- `package.json` — dependencies and scripts

### 3. Modify the home page

Replace the contents of `app/page.tsx` with:

```typescript
export default function Home() {
  const tools = ["TypeScript", "React", "Next.js", "Hono", "Vitest"];

  return (
    <main style={{ padding: "2rem" }}>
      <h1>ML Dashboard</h1>
      <p>Learning JS/TS for AI engineering.</p>

      <h2>Tools Covered</h2>
      <ul>
        {tools.map(tool => (
          <li key={tool}>{tool}</li>
        ))}
      </ul>
    </main>
  );
}
```

Save and check the browser — it should update automatically (hot reload).

### 4. Create an About page

Create `app/about/page.tsx`:

```typescript
export default function About() {
  return (
    <main style={{ padding: "2rem" }}>
      <h1>About</h1>
      <p>This is a learning project for JS/TS.</p>
    </main>
  );
}
```

Visit http://localhost:3000/about — it works without any router configuration.

### 5. Add a navigation layout

Edit `app/layout.tsx` to add a nav bar:

```typescript
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <nav style={{ padding: "1rem", borderBottom: "1px solid #ccc", display: "flex", gap: "1rem" }}>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
        {children}
      </body>
    </html>
  );
}
```

Now both pages share the nav bar. This is the layout system — layouts wrap all pages at or below their level in the file tree.

### 6. Verify understanding

Answer these questions (check your answers by experimenting):
- What happens if you create `app/experiments/page.tsx`? What URL does it map to?
- What happens if you create `app/experiments/layout.tsx`? Which pages does it affect?
- What's the difference between `layout.tsx` and `page.tsx`?

<details>
<summary>Answers</summary>

- `app/experiments/page.tsx` maps to `/experiments`
- `app/experiments/layout.tsx` wraps only pages under `/experiments/*` (not the home page)
- `layout.tsx` persists across navigations (doesn't re-render). `page.tsx` is the actual page content that changes.
</details>
