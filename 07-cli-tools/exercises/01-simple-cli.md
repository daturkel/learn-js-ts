# Exercise: File Stats CLI

Build a CLI tool that analyzes files in a directory.

## Goal

Create a command-line tool using Commander and chalk that scans a directory and reports file statistics.

## Setup

```bash
mkdir file-stats && cd file-stats
npm init -y
npm install commander chalk
npm install -D typescript tsx
```

## Tasks

Build `src/index.ts` that:

1. Takes a directory path as an argument
2. Recursively scans for files
3. Reports: file count by extension, total lines of code, largest files
4. Supports options:
   - `--extension <ext>` — filter to specific extension (e.g., `--extension .ts`)
   - `--top <n>` — show top N largest files (default: 5)
   - `--json` — output as JSON instead of formatted table

### Example usage

```bash
npx tsx src/index.ts ./src
npx tsx src/index.ts ./src --extension .ts --top 3
npx tsx src/index.ts ./src --json
```

### Example output

```
Directory: ./src

Files by extension:
  .ts        12 files    1,847 lines
  .json       3 files       45 lines
  .md         2 files      128 lines

Top 5 largest files:
  1. src/parser.ts          342 lines
  2. src/cli.ts             298 lines
  3. src/utils.ts           187 lines
  4. src/config.ts          156 lines
  5. src/index.ts           134 lines

Total: 17 files, 2,020 lines
```

<details>
<summary>Hint: Recursive file scanning</summary>

```typescript
import { readdir, stat, readFile } from "node:fs/promises";
import path from "node:path";

async function getFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];

  for (const entry of entries) {
    const fullPath = path.join(dir, entry.name);
    if (entry.isDirectory()) {
      if (entry.name === "node_modules" || entry.name === ".git") continue;
      files.push(...await getFiles(fullPath));
    } else {
      files.push(fullPath);
    }
  }

  return files;
}
```
</details>

<details>
<summary>Hint: Counting lines</summary>

```typescript
async function countLines(filePath: string): Promise<number> {
  const content = await readFile(filePath, "utf-8");
  return content.split("\n").length;
}
```
</details>

<details>
<summary>Solution</summary>

```typescript
#!/usr/bin/env node
import { Command } from "commander";
import chalk from "chalk";
import { readdir, readFile } from "node:fs/promises";
import path from "node:path";

// --- File scanning ---

interface FileInfo {
  path: string;
  extension: string;
  lines: number;
}

async function getFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];

  for (const entry of entries) {
    const fullPath = path.join(dir, entry.name);
    if (entry.isDirectory()) {
      if (["node_modules", ".git", "dist", "__pycache__"].includes(entry.name)) continue;
      files.push(...await getFiles(fullPath));
    } else {
      files.push(fullPath);
    }
  }

  return files;
}

async function analyzeFile(filePath: string): Promise<FileInfo> {
  const content = await readFile(filePath, "utf-8");
  return {
    path: filePath,
    extension: path.extname(filePath) || "(no ext)",
    lines: content.split("\n").length,
  };
}

// --- CLI ---

const program = new Command();

program
  .name("file-stats")
  .description("Analyze files in a directory")
  .argument("<directory>", "Directory to scan")
  .option("-e, --extension <ext>", "Filter by extension (e.g., .ts)")
  .option("-t, --top <n>", "Show top N largest files", "5")
  .option("--json", "Output as JSON")
  .action(async (directory, options) => {
    try {
      const allPaths = await getFiles(directory);
      const allFiles = await Promise.all(allPaths.map(analyzeFile));

      // Filter by extension if specified
      const files = options.extension
        ? allFiles.filter(f => f.extension === options.extension)
        : allFiles;

      if (files.length === 0) {
        console.log(chalk.yellow("No files found."));
        return;
      }

      // Group by extension
      const byExt: Record<string, { count: number; lines: number }> = {};
      for (const f of files) {
        const entry = byExt[f.extension] ?? { count: 0, lines: 0 };
        entry.count++;
        entry.lines += f.lines;
        byExt[f.extension] = entry;
      }

      // Top N largest
      const topN = parseInt(options.top);
      const largest = [...files].sort((a, b) => b.lines - a.lines).slice(0, topN);

      // Total
      const totalFiles = files.length;
      const totalLines = files.reduce((sum, f) => sum + f.lines, 0);

      // Output
      if (options.json) {
        console.log(JSON.stringify({ directory, byExtension: byExt, largest, totalFiles, totalLines }, null, 2));
        return;
      }

      console.log(chalk.bold(`\nDirectory: ${directory}\n`));

      console.log(chalk.bold("Files by extension:"));
      const sortedExts = Object.entries(byExt).sort(([, a], [, b]) => b.lines - a.lines);
      for (const [ext, data] of sortedExts) {
        console.log(
          `  ${chalk.cyan(ext.padEnd(12))} ${String(data.count).padStart(4)} files ${chalk.gray(String(data.lines).padStart(8) + " lines")}`
        );
      }

      console.log(chalk.bold(`\nTop ${topN} largest files:`));
      largest.forEach((f, i) => {
        console.log(
          `  ${chalk.gray(`${i + 1}.`)} ${f.path.padEnd(35)} ${chalk.yellow(String(f.lines).padStart(6) + " lines")}`
        );
      });

      console.log(chalk.bold(`\nTotal: ${totalFiles} files, ${totalLines.toLocaleString()} lines\n`));
    } catch (error) {
      console.error(chalk.red(`Error: ${(error as Error).message}`));
      process.exit(1);
    }
  });

program.parse();
```

Run it on this curriculum directory to see real output:
```bash
npx tsx src/index.ts ../../ --extension .md
```
</details>

---

**Next:** [Exercise 2: Interactive CLI →](/07-cli-tools/exercises/02-interactive-cli.md)
