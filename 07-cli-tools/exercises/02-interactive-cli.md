# Exercise: Interactive CLI — ML Project Scaffolder

Build an interactive CLI that helps configure a new ML project.

## Goal

Practice Inquirer prompts by building a tool that asks questions and generates a config file.

## Setup

```bash
mkdir ml-scaffolder && cd ml-scaffolder
npm init -y
npm install @inquirer/prompts chalk
npm install -D typescript tsx
```

## Requirements

Create `src/index.ts` that:

1. Asks the user a series of questions:
   - **Project name** (text input, validate non-empty)
   - **ML framework**: PyTorch, TensorFlow, JAX (select one)
   - **Task type**: Classification, Generation, Regression, Other (select one)
   - **Include data pipeline?** (yes/no)
   - **Experiment tracking**: MLflow, Weights & Biases, None (select one)
   - **Extra features**: Docker, CI/CD, pre-commit hooks, DVC (select multiple)

2. Shows a colored summary of selections before writing

3. Writes a `project-config.json` file with the choices

### Example session

```
ML Project Scaffolder

? Project name: sentiment-analysis
? ML framework: PyTorch
? Task type: Classification
? Include data pipeline? Yes
? Experiment tracking: Weights & Biases
? Extra features: Docker, CI/CD

── Summary ──────────────────────────
  Project:    sentiment-analysis
  Framework:  PyTorch
  Task:       Classification
  Pipeline:   Yes
  Tracking:   Weights & Biases
  Features:   Docker, CI/CD

Write project-config.json? Yes
✓ Config written to project-config.json
```

<details>
<summary>Hint: Input validation</summary>

```typescript
import { input } from "@inquirer/prompts";

const name = await input({
  message: "Project name:",
  validate: (value) => {
    if (!value.trim()) return "Project name is required";
    if (!/^[a-z0-9-]+$/.test(value)) return "Use lowercase letters, numbers, and hyphens only";
    return true;
  },
});
```
</details>

<details>
<summary>Solution</summary>

```typescript
import { input, select, confirm, checkbox } from "@inquirer/prompts";
import chalk from "chalk";
import { writeFile } from "node:fs/promises";

console.log(chalk.bold("\nML Project Scaffolder\n"));

// --- Prompts ---

const projectName = await input({
  message: "Project name:",
  validate: (value) => {
    if (!value.trim()) return "Project name is required";
    if (!/^[a-z0-9-]+$/i.test(value)) return "Use letters, numbers, and hyphens only";
    return true;
  },
});

const framework = await select({
  message: "ML framework:",
  choices: [
    { name: "PyTorch", value: "pytorch" },
    { name: "TensorFlow", value: "tensorflow" },
    { name: "JAX", value: "jax" },
  ],
});

const taskType = await select({
  message: "Task type:",
  choices: [
    { name: "Classification", value: "classification" },
    { name: "Generation", value: "generation" },
    { name: "Regression", value: "regression" },
    { name: "Other", value: "other" },
  ],
});

const includePipeline = await confirm({
  message: "Include data pipeline?",
  default: true,
});

const tracking = await select({
  message: "Experiment tracking:",
  choices: [
    { name: "Weights & Biases", value: "wandb" },
    { name: "MLflow", value: "mlflow" },
    { name: "None", value: "none" },
  ],
});

const features = await checkbox({
  message: "Extra features:",
  choices: [
    { name: "Docker", value: "docker" },
    { name: "CI/CD pipeline", value: "ci" },
    { name: "Pre-commit hooks", value: "hooks" },
    { name: "Data versioning (DVC)", value: "dvc" },
  ],
});

// --- Summary ---

const frameworkNames: Record<string, string> = {
  pytorch: "PyTorch", tensorflow: "TensorFlow", jax: "JAX",
};
const trackingNames: Record<string, string> = {
  wandb: "Weights & Biases", mlflow: "MLflow", none: "None",
};

console.log(chalk.bold("\n── Summary ──────────────────────────"));
console.log(`  ${chalk.gray("Project:")}    ${chalk.cyan(projectName)}`);
console.log(`  ${chalk.gray("Framework:")}  ${frameworkNames[framework]}`);
console.log(`  ${chalk.gray("Task:")}       ${taskType}`);
console.log(`  ${chalk.gray("Pipeline:")}   ${includePipeline ? "Yes" : "No"}`);
console.log(`  ${chalk.gray("Tracking:")}   ${trackingNames[tracking]}`);
console.log(`  ${chalk.gray("Features:")}   ${features.length > 0 ? features.join(", ") : "None"}`);
console.log();

// --- Write config ---

const shouldWrite = await confirm({
  message: "Write project-config.json?",
  default: true,
});

if (shouldWrite) {
  const config = {
    name: projectName,
    framework,
    taskType,
    dataPipeline: includePipeline,
    experimentTracking: tracking,
    features,
    createdAt: new Date().toISOString(),
  };

  await writeFile("project-config.json", JSON.stringify(config, null, 2) + "\n");
  console.log(chalk.green("✓ Config written to project-config.json"));
} else {
  console.log(chalk.yellow("Cancelled."));
}
```
</details>

## Stretch Goal

Instead of just writing a JSON config, generate actual project files:
- `README.md` with the project name and framework
- `.gitignore` with Python and framework-specific entries
- A `Dockerfile` if Docker was selected

<details>
<summary>Stretch solution sketch</summary>

```typescript
import { mkdir } from "node:fs/promises";

await mkdir(projectName, { recursive: true });

// README
await writeFile(`${projectName}/README.md`, `# ${projectName}\n\nBuilt with ${frameworkNames[framework]}.\n`);

// .gitignore
const ignoreLines = [
  "node_modules/", "__pycache__/", "*.pyc", ".env",
  "dist/", "*.egg-info/", ".venv/",
];
if (framework === "pytorch") ignoreLines.push("*.pt", "*.pth");
await writeFile(`${projectName}/.gitignore`, ignoreLines.join("\n") + "\n");

// Dockerfile
if (features.includes("docker")) {
  const baseImage = framework === "pytorch" ? "pytorch/pytorch:latest" : "python:3.11-slim";
  await writeFile(`${projectName}/Dockerfile`, `FROM ${baseImage}\nWORKDIR /app\nCOPY . .\nRUN pip install -r requirements.txt\n`);
}

console.log(chalk.green(`✓ Project created in ./${projectName}/`));
```
</details>
