# Module 07: CLI Tools

Build command-line tools in TypeScript using Commander and Inquirer.

**Prerequisites:** [Module 06](../06-testing/README.md)
**Time:** 1-2 sessions

## Why Build CLIs in JS/TS?

Many AI developer tools are Node CLIs: Claude Code, Vercel CLI, Wrangler, etc. The npm ecosystem makes it easy to build, distribute, and install CLI tools.

## Commander.js — Argument Parsing

Commander is like Python's Click — it handles commands, options, and arguments with automatic help generation.

```bash
npm install commander
```

### Basic CLI

```python
# Python (Click)
import click

@click.command()
@click.argument("name")
@click.option("--greeting", default="Hello", help="The greeting to use")
def hello(name, greeting):
    """Greet someone."""
    click.echo(f"{greeting}, {name}!")

if __name__ == "__main__":
    hello()
```

```typescript
// TypeScript (Commander)
import { Command } from "commander";

const program = new Command();

program
  .name("hello")
  .description("Greet someone")
  .argument("<name>", "Person to greet")
  .option("-g, --greeting <word>", "The greeting to use", "Hello")
  .action((name, options) => {
    console.log(`${options.greeting}, ${name}!`);
  });

program.parse();
```

Run: `npx tsx hello.ts World` → `Hello, World!`
Run: `npx tsx hello.ts World -g Hi` → `Hi, World!`

### Subcommands

```typescript
import { Command } from "commander";

const program = new Command();
program.name("ml").description("ML experiment CLI");

program
  .command("train")
  .description("Train a model")
  .requiredOption("-m, --model <name>", "Model name")
  .option("-e, --epochs <n>", "Number of epochs", "10")
  .action((options) => {
    console.log(`Training ${options.model} for ${options.epochs} epochs`);
  });

program
  .command("evaluate")
  .description("Evaluate a model")
  .argument("<model>", "Model to evaluate")
  .option("--dataset <name>", "Dataset to use", "test")
  .action((model, options) => {
    console.log(`Evaluating ${model} on ${options.dataset}`);
  });

program.parse();
```

Run: `npx tsx ml.ts train -m bert -e 5`
Run: `npx tsx ml.ts evaluate bert --dataset validation`

## Inquirer — Interactive Prompts

`@inquirer/prompts` provides interactive prompts — like Python's `questionary`:

```bash
npm install @inquirer/prompts
```

```typescript
import { input, select, confirm, checkbox } from "@inquirer/prompts";

const name = await input({ message: "Project name:" });

const framework = await select({
  message: "ML framework:",
  choices: [
    { name: "PyTorch", value: "pytorch" },
    { name: "TensorFlow", value: "tensorflow" },
    { name: "JAX", value: "jax" },
  ],
});

const useDocker = await confirm({ message: "Include Dockerfile?" });

const features = await checkbox({
  message: "Features:",
  choices: [
    { name: "CI/CD pipeline", value: "ci" },
    { name: "Pre-commit hooks", value: "hooks" },
    { name: "Data versioning (DVC)", value: "dvc" },
  ],
});

console.log({ name, framework, useDocker, features });
```

## Terminal Output

### Colors with chalk

```bash
npm install chalk
```

```typescript
import chalk from "chalk";

console.log(chalk.green("Success!"));
console.log(chalk.red.bold("Error: something failed"));
console.log(chalk.blue(`Processing ${chalk.bold("data.csv")}...`));
console.log(chalk.gray("Debug: loaded config"));
```

### Spinners with ora

```bash
npm install ora
```

```typescript
import ora from "ora";

const spinner = ora("Processing data...").start();
await longRunningTask();
spinner.succeed("Done!");     // Green checkmark
// or spinner.fail("Error");  // Red X
```

## Packaging a CLI

### 1. Add a `bin` field to `package.json`

```json
{
  "name": "my-cli",
  "bin": {
    "my-cli": "dist/index.js"
  }
}
```

### 2. Add a shebang to your entry file

```typescript
#!/usr/bin/env node
import { Command } from "commander";
// ... rest of your CLI
```

### 3. Build and link

```bash
npm run build          # Compile TS to JS
npm link               # Creates global symlink
my-cli --help          # Now works as a command
```

### Distribution

- **npm publish** — users install with `npm install -g my-cli`
- **npx** — users run without installing: `npx my-cli`
- Compare to Python: `pip install my-cli` or `pipx run my-cli`

## Running TypeScript Directly

During development, use `tsx` to skip the compilation step:

```bash
npm install -D tsx
npx tsx src/index.ts           # Run directly
npx tsx --watch src/index.ts   # Watch mode
```

This is like running `python script.py` — no build step needed.

## Further Reading

- [Commander.js](https://github.com/tj/commander.js) — Full documentation
- [Inquirer.js](https://github.com/SBoudrias/Inquirer.js) — All prompt types
- [Node.js CLI Best Practices](https://github.com/lirantal/nodejs-cli-apps-best-practices)

## Exercises

1. [Simple CLI](./exercises/01-simple-cli.md) — Build a file stats tool
2. [Interactive CLI](./exercises/02-interactive-cli.md) — Build a project scaffolder

Next: [Module 08 — Web Servers and APIs](../08-web-servers-and-apis/README.md)
