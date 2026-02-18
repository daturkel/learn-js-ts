# Exercise: Bundle for Distribution

Bundle a TypeScript CLI tool into a single executable file that can run without `npm install`.

## Setup

```bash
mkdir bundled-cli && cd bundled-cli
npm init -y
npm install commander chalk
npm install -D typescript tsx esbuild
```

Add to `package.json`: `"type": "module"`

## Requirements

### 1. Create a CLI tool

Create `src/cli.ts` — a simple file statistics tool:

```typescript
import { Command } from "commander";
import chalk from "chalk";
import { readdir, stat, readFile } from "node:fs/promises";
import { join, extname } from "node:path";

const program = new Command();

program
  .name("fstat")
  .description("File statistics tool")
  .argument("<directory>", "Directory to analyze")
  .option("-e, --extension <ext>", "Filter by extension (e.g. .ts)")
  .action(async (directory: string, options: { extension?: string }) => {
    const files = await getFiles(directory);
    const filtered = options.extension
      ? files.filter(f => extname(f) === options.extension)
      : files;

    let totalLines = 0;
    let totalSize = 0;
    const byCounts: Record<string, number> = {};

    for (const file of filtered) {
      const info = await stat(file);
      totalSize += info.size;
      const ext = extname(file) || "(no ext)";
      byCounts[ext] = (byCounts[ext] ?? 0) + 1;

      const content = await readFile(file, "utf-8");
      totalLines += content.split("\n").length;
    }

    console.log(chalk.bold(`\nResults for ${directory}:\n`));
    console.log(`  Files: ${chalk.cyan(filtered.length)}`);
    console.log(`  Lines: ${chalk.cyan(totalLines)}`);
    console.log(`  Size:  ${chalk.cyan(formatBytes(totalSize))}`);
    console.log(`\n  By extension:`);
    for (const [ext, count] of Object.entries(byCounts).sort((a, b) => b[1] - a[1])) {
      console.log(`    ${ext}: ${count}`);
    }
  });

async function getFiles(dir: string): Promise<string[]> {
  const entries = await readdir(dir, { withFileTypes: true });
  const files: string[] = [];
  for (const entry of entries) {
    const full = join(dir, entry.name);
    if (entry.isDirectory() && entry.name !== "node_modules" && !entry.name.startsWith(".")) {
      files.push(...await getFiles(full));
    } else if (entry.isFile()) {
      files.push(full);
    }
  }
  return files;
}

function formatBytes(bytes: number): string {
  if (bytes < 1024) return `${bytes} B`;
  if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
  return `${(bytes / (1024 * 1024)).toFixed(1)} MB`;
}

program.parse();
```

Test it works:
```bash
npx tsx src/cli.ts .
npx tsx src/cli.ts . --extension .ts
```

### 2. Bundle with esbuild

Bundle everything into a single file:

```bash
npx esbuild src/cli.ts \
  --bundle \
  --platform=node \
  --target=node20 \
  --outfile=dist/fstat.js \
  --format=esm \
  --banner:js='#!/usr/bin/env node'
```

Test the bundled version:
```bash
node dist/fstat.js .
```

It should work identically to the `tsx` version but without needing `node_modules`.

### 3. Make it executable

```bash
chmod +x dist/fstat.js
./dist/fstat.js .
```

The `#!/usr/bin/env node` banner (shebang) tells the OS to run it with Node.js.

### 4. Compare file sizes

Check how big the bundle is:

```bash
# Bundled file size
ls -lh dist/fstat.js

# vs total node_modules size
du -sh node_modules
```

The bundle is typically a few hundred KB vs tens of MB for node_modules.

### 5. Add a build script

Add to `package.json`:

```json
{
  "scripts": {
    "dev": "tsx src/cli.ts",
    "build": "esbuild src/cli.ts --bundle --platform=node --target=node20 --outfile=dist/fstat.js --format=esm --banner:js='#!/usr/bin/env node'",
    "start": "node dist/fstat.js"
  },
  "bin": {
    "fstat": "dist/fstat.js"
  }
}
```

Now:
```bash
npm run build   # Bundle
npm start -- .  # Run bundled version (-- passes args through)
```

### 6. Try Bun bundler (stretch)

If you have Bun installed:

```bash
bun build src/cli.ts --outfile=dist/fstat-bun.js --target=node

# Compare sizes
ls -lh dist/fstat.js dist/fstat-bun.js
```

<details>
<summary>Hint: Common esbuild issues</summary>

**"Dynamic require" warning:** Some packages use `require()` dynamically. Add `--external:package-name` to exclude them from the bundle.

**Node.js built-in modules:** esbuild automatically handles `node:fs`, `node:path`, etc. — they stay as imports since they're provided by the runtime.

**ESM vs CJS:** If you get module format errors, try `--format=cjs` instead of `--format=esm`.

</details>

<details>
<summary>Solution</summary>

The code is provided in the requirements above. Here's the complete `package.json`:

```json
{
  "name": "bundled-cli",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "fstat": "dist/fstat.js"
  },
  "scripts": {
    "dev": "tsx src/cli.ts",
    "build": "esbuild src/cli.ts --bundle --platform=node --target=node20 --outfile=dist/fstat.js --format=esm --banner:js='#!/usr/bin/env node'",
    "start": "node dist/fstat.js"
  },
  "dependencies": {
    "commander": "^12.0.0",
    "chalk": "^5.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "tsx": "^4.0.0",
    "esbuild": "^0.20.0"
  }
}
```

### Build and test

```bash
# Build
npm run build

# Test
node dist/fstat.js .
node dist/fstat.js . --extension .ts
node dist/fstat.js . --extension .json

# Check bundle size
ls -lh dist/fstat.js
# Typically ~200-400 KB including commander and chalk
```

### What bundling does

Before bundling:
```
src/cli.ts
  → imports commander (node_modules/commander/...)
  → imports chalk (node_modules/chalk/...)
  → imports node:fs, node:path (built-in)
```

After bundling:
```
dist/fstat.js (single file)
  → commander code inlined
  → chalk code inlined
  → node:fs, node:path remain as imports (provided by runtime)
```

### Why bundle?

| Without bundling | With bundling |
|-----------------|---------------|
| Need `npm install` first | Just copy one file |
| `node_modules` = 30+ MB | Bundle = ~300 KB |
| Needs `tsx` for TypeScript | Plain JavaScript |
| Multiple files to manage | Single file |

### Python comparison

This is similar to:
- `pyinstaller` — bundles Python into a standalone executable
- `zipapp` — packages Python code into a `.pyz` file

But JS bundling is simpler because there's no interpreter to embed — Node.js is expected to be installed.

### Distribution options

After bundling, you can:
1. **Share the file** — others run it with `node dist/fstat.js`
2. **Publish to npm** — `npm publish`, then `npx fstat`
3. **Link locally** — `npm link`, then use `fstat` anywhere on your machine

</details>
