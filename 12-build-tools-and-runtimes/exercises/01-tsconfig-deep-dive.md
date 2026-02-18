# Exercise: TSConfig Deep Dive

Explore TypeScript compiler settings by creating a project and experimenting with different configurations.

## Setup

```bash
mkdir tsconfig-lab && cd tsconfig-lab
npm init -y
npm install -D typescript
```

Add to `package.json`: `"type": "module"`

## Requirements

### 1. Create a strict TypeScript project

Create `tsconfig.json` with:
- Output to `dist/`
- Source in `src/`
- ES2022 target
- Node16 module system
- `strict: true`
- Source maps enabled

Create `src/index.ts` with this code:

```typescript
interface User {
  name: string;
  email: string;
  age?: number;
}

function greet(user: User): string {
  return `Hello, ${user.name} (${user.email})`;
}

const users: User[] = [
  { name: "Alice", email: "alice@example.com", age: 30 },
  { name: "Bob", email: "bob@example.com" },
];

for (const user of users) {
  console.log(greet(user));
  if (user.age) {
    console.log(`  Age: ${user.age}`);
  }
}
```

Compile with `npx tsc` and examine the output in `dist/`.

### 2. Explore strictness levels

Add this deliberately questionable code to a new file `src/loose.ts`:

```typescript
function processData(data: any) {
  return data.value.toString();
}

function findUser(users: User[], name: string) {
  const user = users.find(u => u.name === name);
  console.log(user.email);  // What if user is undefined?
}

const config: Record<string, string> = {};
console.log(config["missing"].toUpperCase());  // What if key doesn't exist?
```

Try these settings one at a time and see which errors appear:
- `"noImplicitAny": true` — what does it catch?
- `"strictNullChecks": true` — what does it catch?
- `"noUncheckedIndexedAccess": true` — what does it catch?

Fix each error the compiler finds.

### 3. Path aliases

Configure path aliases so you can write:
```typescript
import { greet } from "@/utils/greet";  // instead of "../../../utils/greet"
```

Add to `tsconfig.json`:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

Create `src/utils/greet.ts` and import it from `src/index.ts` using the alias.

### 4. Declaration files

Enable `"declaration": true` and compile. Look at the `.d.ts` files generated in `dist/`. These are like Python's `.pyi` stub files — they describe the types without the implementation.

### 5. Compare configurations

Create two tsconfig files:
- `tsconfig.strict.json` — maximum strictness
- `tsconfig.loose.json` — minimal strictness

Run `npx tsc -p tsconfig.strict.json` and `npx tsc -p tsconfig.loose.json` on the same code. How many more errors does the strict config catch?

<details>
<summary>Hint: Maximum strictness config</summary>

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noPropertyAccessFromIndexSignature": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true
  }
}
```

</details>

<details>
<summary>Solution</summary>

**`tsconfig.json`**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "esModuleInterop": true,
    "declaration": true,
    "sourceMap": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

**`src/loose.ts`** — fixed version:

```typescript
interface User {
  name: string;
  email: string;
  age?: number;
}

// Fix 1: Type the parameter (no `any`)
function processData(data: { value: unknown }): string {
  return String(data.value);
}

// Fix 2: Handle the undefined case
function findUser(users: User[], name: string): void {
  const user = users.find(u => u.name === name);
  if (user) {
    console.log(user.email);  // now TypeScript knows user is defined
  } else {
    console.log(`User "${name}" not found`);
  }
}

// Fix 3: Check for undefined before using
const config: Record<string, string> = {};
const value = config["missing"];
if (value !== undefined) {
  console.log(value.toUpperCase());
}
```

**`tsconfig.strict.json`**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noPropertyAccessFromIndexSignature": true
  },
  "include": ["src"]
}
```

**`tsconfig.loose.json`**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "dist",
    "rootDir": "src",
    "strict": false
  },
  "include": ["src"]
}
```

### What each setting catches

| Setting | What it catches | Python equivalent |
|---------|----------------|-------------------|
| `strict` | All strict checks enabled | `mypy --strict` |
| `noImplicitAny` | Untyped parameters/variables | `mypy --disallow-untyped-defs` |
| `strictNullChecks` | Using values that might be `null`/`undefined` | `mypy` with `Optional` checks |
| `noUncheckedIndexedAccess` | Array/object access that might be undefined | No direct equivalent |
| `noImplicitReturns` | Functions that don't return on all paths | `mypy --warn-no-return` |

### Compiled output

When you run `npx tsc`, the `dist/` folder contains:
- `index.js` — compiled JavaScript (your code without type annotations)
- `index.d.ts` — type declarations (just the types, for consumers of your package)
- `index.js.map` — source maps (maps compiled JS back to your TS for debugging)

```bash
ls dist/
# index.d.ts  index.js  index.js.map  utils/greet.d.ts  utils/greet.js  utils/greet.js.map
```

</details>
