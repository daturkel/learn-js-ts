# Environment Setup

One-time setup to get your development environment ready.

## 1. Install Node.js

Node.js is the JavaScript runtime (like CPython is to Python). Install version 22 or later.

**Recommended: use a version manager** (like `pyenv` for Python):

```bash
# Install fnm (Fast Node Manager)
curl -fsSL https://fnm.vercel.app/install | bash

# Install Node.js 22
fnm install 22
fnm use 22

# Verify
node --version   # Should print v22.x.x
npm --version    # Should print 10.x.x
```

**Alternative: direct install** from [nodejs.org](https://nodejs.org/).

### What you just installed

- `node` — the JavaScript runtime (like `python3`)
- `npm` — the package manager (like `pip`)
- `npx` — runs packages without installing them globally (like `pipx run`)

## 2. Install TypeScript and tsx

TypeScript is used from Module 02 onward. Install both globally:

```bash
npm install -g typescript tsx

# Verify
tsc --version   # Should print Version 5.x.x
tsx --version   # Should print a version number
```

- **`tsc`** — the TypeScript compiler. Use `tsc --noEmit` to type-check without producing output files.
- **`tsx`** — runs TypeScript files directly, like `python` runs `.py` files. Use this to execute `.ts` files.

Alternatively, you can install both per-project with `npm install -D typescript tsx` and run them via `npx tsc` / `npx tsx`. Either way works — just make sure `typescript` is installed before running `tsc`, because there's an unrelated package called `tsc` on npm that `npx` will grab otherwise.

## 3. Editor Setup

**VS Code** is strongly recommended for TypeScript development. The TypeScript integration is excellent out of the box.

If you use VS Code, no additional extensions are strictly required — TypeScript support is built in. Optionally install:

- **ESLint** — linting (like `flake8`/`ruff` for Python)
- **Prettier** — auto-formatting (like `black` for Python)

If you use another editor (Cursor, Zed, Neovim), ensure it has TypeScript language server support.

## 4. Verify Your Setup

Create a test file and run it:

```bash
# Create a simple JS file
echo 'console.log("Hello from Node.js!")' > test.js

# Run it
node test.js

# Clean up
rm test.js
```

You should see `Hello from Node.js!` printed.

## 5. API Keys (needed from Module 10 onward)

Modules 10-11 use AI provider APIs. You'll need at least one of:

- **Anthropic API key**: Sign up at [console.anthropic.com](https://console.anthropic.com/)
- **OpenAI API key**: Sign up at [platform.openai.com](https://platform.openai.com/)

Set them as environment variables when you get to those modules:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
```

Or add them to a `.env` file (you'll learn how in Module 05).

## 6. Optional: Install Bun (for Module 12)

Bun is an alternative JavaScript runtime that's faster and has built-in TypeScript support. You'll explore it in Module 12, but you can install it now if you want:

```bash
curl -fsSL https://bun.sh/install | bash
bun --version
```

## Ready?

Head to [Module 00: JS in 5 Minutes](./00-js-in-5-minutes/README.md) to get started.
