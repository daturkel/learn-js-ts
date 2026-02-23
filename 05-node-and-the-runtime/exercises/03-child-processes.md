# Exercise: Child Processes — Git Log Analyzer

Spawn `git log` as a subprocess and analyze the output.

## Goal

Practice `child_process` by parsing real git history. This is the same kind of task you'd do with `subprocess.run()` in Python.

## Setup

Run this exercise from any git repository (or this curriculum repo itself if you've initialized it with `git init`).

Create `git-stats.ts`.

## Tasks

### 1. Get git log output

Use `execSync` or promisified `exec` to run:

```bash
git log --format="%H|%an|%ae|%ad|%s" --date=short
```

This outputs one commit per line: `hash|author name|email|date|subject`

### 2. Parse into typed objects

```typescript
interface Commit {
  hash: string;
  author: string;
  email: string;
  date: string;
  subject: string;
}
```

Parse each line into a `Commit` object.

### 3. Compute statistics

- Total commits
- Unique authors (and commit count per author)
- Most active day of the week
- Commits per month
- Most common words in commit subjects (excluding common words like "the", "a", "and")

### 4. Handle errors

- What if the current directory isn't a git repo? Catch the error and print a helpful message.
- What if git isn't installed?

<details>
<summary>Hint: Error handling for non-git directories</summary>

```typescript
try {
  execSync("git rev-parse --is-inside-work-tree", { encoding: "utf-8" });
} catch {
  console.error("Error: Not a git repository");
  process.exit(1);
}
```
</details>

<details>
<summary>Solution</summary>

```typescript
import { execSync } from "node:child_process";

// --- Types ---

interface Commit {
  hash: string;
  author: string;
  email: string;
  date: string;
  subject: string;
}

interface GitStats {
  totalCommits: number;
  authors: Record<string, number>;
  commitsByDay: Record<string, number>;
  commitsByMonth: Record<string, number>;
  topWords: [string, number][];
}

// --- Check if we're in a git repo ---

try {
  execSync("git rev-parse --is-inside-work-tree", {
    encoding: "utf-8",
    stdio: "pipe",  // Suppress stderr
  });
} catch {
  console.error("Error: Not inside a git repository.");
  console.error("Run this script from within a git repo.");
  process.exit(1);
}

// --- Get git log ---

const logOutput = execSync(
  'git log --format="%H|%an|%ae|%ad|%s" --date=short',
  { encoding: "utf-8", maxBuffer: 10 * 1024 * 1024 }  // 10MB buffer for large repos
);

// --- Parse commits ---

const commits: Commit[] = logOutput
  .trim()
  .split("\n")
  .filter(line => line.length > 0)
  .map(line => {
    const [hash, author, email, date, ...subjectParts] = line.split("|");
    return {
      hash,
      author,
      email,
      date,
      subject: subjectParts.join("|"),  // Subject might contain |
    };
  });

// --- Compute statistics ---

// Commits per author
const authors: Record<string, number> = {};
for (const c of commits) {
  authors[c.author] = (authors[c.author] ?? 0) + 1;
}

// Commits by day of week
const dayNames = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
const commitsByDay: Record<string, number> = {};
for (const c of commits) {
  const day = dayNames[new Date(c.date).getDay()];
  commitsByDay[day] = (commitsByDay[day] ?? 0) + 1;
}

// Commits by month
const commitsByMonth: Record<string, number> = {};
for (const c of commits) {
  const month = c.date.slice(0, 7);  // "2024-01"
  commitsByMonth[month] = (commitsByMonth[month] ?? 0) + 1;
}

// Top words in commit subjects
const stopWords = new Set(["the", "a", "an", "and", "or", "in", "to", "for", "of", "is", "it", "on", "at", "by"]);
const wordCounts: Record<string, number> = {};
for (const c of commits) {
  const words = c.subject.toLowerCase().split(/\W+/).filter(w => w.length > 2 && !stopWords.has(w));
  for (const word of words) {
    wordCounts[word] = (wordCounts[word] ?? 0) + 1;
  }
}
const topWords = Object.entries(wordCounts)
  .sort(([, a], [, b]) => b - a)
  .slice(0, 10);

// --- Output ---

console.log(`\nGit Repository Statistics`);
console.log(`${"=".repeat(40)}`);
console.log(`Total commits: ${commits.length}`);

console.log(`\nAuthors (${Object.keys(authors).length}):`);
const sortedAuthors = Object.entries(authors).sort(([, a], [, b]) => b - a);
for (const [name, count] of sortedAuthors.slice(0, 10)) {
  const bar = "#".repeat(Math.min(count, 40));
  console.log(`  ${name.padEnd(25)} ${String(count).padStart(5)} ${bar}`);
}

console.log(`\nCommits by day of week:`);
for (const day of dayNames) {
  const count = commitsByDay[day] ?? 0;
  const bar = "#".repeat(Math.round(count / Math.max(...Object.values(commitsByDay)) * 20));
  console.log(`  ${day.padEnd(12)} ${String(count).padStart(5)} ${bar}`);
}

console.log(`\nTop 10 words in commit messages:`);
for (const [word, count] of topWords) {
  console.log(`  ${word.padEnd(20)} ${count}`);
}

if (commits.length > 0) {
  console.log(`\nFirst commit: ${commits[commits.length - 1].date}`);
  console.log(`Latest commit: ${commits[0].date}`);
}
```

### Python equivalent

```python
import subprocess
from collections import Counter

result = subprocess.run(
    ["git", "log", '--format=%H|%an|%ae|%ad|%s', "--date=short"],
    capture_output=True, text=True
)

commits = []
for line in result.stdout.strip().split("\n"):
    hash, author, email, date, *subject = line.split("|")
    commits.append({"hash": hash, "author": author, "date": date, "subject": "|".join(subject)})

authors = Counter(c["author"] for c in commits)
```

The Node version is longer because:
- No `Counter` equivalent (build with reduce or manual loop)
- String padding uses `.padEnd()` / `.padStart()` instead of f-string formatting
- `execSync` works like `subprocess.run()` with `capture_output=True`
</details>

---

**Next:** [Module 06: Testing →](/06-testing/README.md)
