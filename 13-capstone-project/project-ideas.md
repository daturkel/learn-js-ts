# Capstone Project Ideas

Choose one of these projects, or design your own. Each combines multiple skills from the curriculum.

---

## Project 1: AI Chat CLI

A terminal-based chat application that talks to Claude or GPT, with conversation history and tool use.

**Skills used:** TypeScript, async/streaming, CLI tools, AI SDKs, file I/O, testing

### Features

**Core:**
- Interactive chat loop in the terminal (type a message, get a streaming response)
- Conversation history maintained across messages
- Save/load conversations to/from JSON files
- System prompt selection (choose a persona)

**Extended:**
- Tool use — give the model tools like `run_shell_command`, `read_file`, `web_search`
- Multiple provider support (Claude and GPT, switchable with a flag)
- Conversation search — search through saved conversations by keyword
- Token usage tracking and cost estimation

**Stretch:**
- MCP server that exposes your saved conversations as resources
- Markdown rendering in the terminal (bold, code blocks, lists)

### Suggested structure

```
ai-chat/
├── src/
│   ├── cli.ts           # Commander setup, main entry point
│   ├── chat.ts          # Chat loop logic
│   ├── providers/
│   │   ├── anthropic.ts # Anthropic SDK wrapper
│   │   └── openai.ts    # OpenAI SDK wrapper
│   ├── tools/           # Tool definitions and implementations
│   ├── storage.ts       # Save/load conversations
│   └── types.ts         # Shared TypeScript interfaces
├── tests/
│   ├── chat.test.ts
│   └── storage.test.ts
└── package.json
```

---

## Project 2: Experiment Dashboard

A full-stack web app for tracking ML experiments, built with Next.js and Hono.

**Skills used:** TypeScript, React/Next.js, Hono API, async, testing, data handling

### Features

**Core:**
- Dashboard page showing all experiments in a table
- Create new experiments via a form
- Experiment detail page with metrics and charts
- Filter and sort experiments by status, model, accuracy

**Extended:**
- Comparison view — select two experiments and compare them side by side
- Import experiments from JSON/CSV files
- Charts for metric visualization (use a library like Recharts or Chart.js)
- API validation with Zod

**Stretch:**
- AI analysis — send experiment data to Claude and get suggestions
- MCP server for experiment data (so Claude Desktop can query your experiments)
- Export to CSV/PDF

### Suggested structure

```
experiment-dashboard/
├── app/
│   ├── page.tsx                  # Dashboard home
│   ├── layout.tsx                # Root layout with nav
│   ├── experiments/
│   │   ├── page.tsx              # Experiment list
│   │   └── [id]/page.tsx         # Experiment detail
│   ├── compare/page.tsx          # Comparison view
│   ├── api/
│   │   └── experiments/route.ts  # REST API
│   └── components/
│       ├── experiment-table.tsx
│       ├── experiment-form.tsx
│       └── metric-chart.tsx
├── lib/
│   ├── types.ts
│   ├── storage.ts                # In-memory or file-based storage
│   └── validation.ts             # Zod schemas
├── tests/
└── package.json
```

---

## Project 3: Document Q&A Tool

A CLI tool that ingests documents and lets you ask questions about them using an AI model.

**Skills used:** TypeScript, async, file I/O, AI SDKs, CLI tools, streaming, testing

### Features

**Core:**
- Ingest text/markdown files from a directory
- Ask questions about the ingested documents
- The tool sends relevant document chunks + the question to Claude
- Streaming response in the terminal

**Extended:**
- Simple keyword-based search to find relevant chunks (no vector DB needed — just TF-IDF or keyword matching)
- Support for multiple file formats (`.txt`, `.md`, `.json`)
- Conversation mode — follow-up questions with context
- Source citations — show which document chunks were used

**Stretch:**
- Watch mode — automatically re-ingest when files change
- MCP server so Claude Desktop can query your documents
- Web UI with Next.js

### Suggested structure

```
doc-qa/
├── src/
│   ├── cli.ts           # CLI entry point
│   ├── ingest.ts        # Read and chunk documents
│   ├── search.ts        # Find relevant chunks
│   ├── ask.ts           # Send to AI and stream response
│   ├── types.ts
│   └── mcp-server.ts    # Optional MCP interface
├── tests/
│   ├── ingest.test.ts
│   └── search.test.ts
└── package.json
```

---

## Project 4: Dev Toolbox MCP Server

An MCP server that provides useful developer tools to AI assistants like Claude Desktop.

**Skills used:** TypeScript, MCP, Node.js APIs, child processes, file I/O, Zod, testing

### Features

**Core tools:**
- `git_status` — get current git status
- `git_log` — get recent commits with filtering options
- `file_stats` — analyze a directory (file count, line count, by language)
- `run_tests` — execute test suite and return results
- `search_code` — search for patterns in code files

**Extended tools:**
- `npm_outdated` — check for outdated dependencies
- `todo_scan` — find TODO/FIXME/HACK comments in codebase
- `env_check` — verify required environment variables are set
- `port_check` — check if a port is in use

**Resources:**
- `project://structure` — current project's file tree
- `project://package-json` — parsed package.json
- `project://readme` — project's README content

**Stretch:**
- Prompt templates for common dev tasks (code review, refactoring, debugging)
- Configuration file to customize which tools are enabled
- Tests for each tool

### Suggested structure

```
dev-toolbox/
├── src/
│   ├── server.ts        # MCP server setup
│   ├── tools/
│   │   ├── git.ts       # Git-related tools
│   │   ├── files.ts     # File analysis tools
│   │   ├── npm.ts       # Package management tools
│   │   └── code.ts      # Code search tools
│   ├── resources/       # Resource providers
│   └── types.ts
├── tests/
│   ├── git.test.ts
│   └── files.test.ts
└── package.json
```

---

## Design Your Own

If none of these appeal, here are some other ideas to riff on:

- **Markdown note-taking app** — CLI + web UI, with AI-powered search and summarization
- **API monitoring tool** — Periodically ping endpoints, track uptime, alert on failures
- **Code review assistant** — Reads git diffs, sends to Claude for review feedback
- **Dataset explorer** — Load CSV/JSON datasets, compute statistics, ask AI to explain patterns
- **Personal knowledge base** — Save and search bookmarks/notes, with AI-powered tagging

Whatever you choose, keep it scoped to something you can build in 3-5 focused sessions. You can always expand later.
