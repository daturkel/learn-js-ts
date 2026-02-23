# Module 11: MCP Servers

Build Model Context Protocol (MCP) servers that give AI models access to your tools and data.


## What is MCP?

The Model Context Protocol (MCP) is an open standard for connecting AI models to external tools and data sources. Instead of each AI app implementing its own tool integrations, MCP provides a universal protocol.

Think of it like this:
- **Without MCP:** Every AI app (Claude, ChatGPT, Cursor) needs custom code to connect to each tool
- **With MCP:** Build one MCP server, and any MCP-compatible client can use it

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│  Claude  │     │  Cursor  │     │ Your App │
│ Desktop  │     │          │     │          │
└────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                │
     └───────┬────────┴────────┬───────┘
             │    MCP Protocol │
             ▼                 ▼
      ┌──────────┐      ┌──────────┐
      │ Your MCP │      │ Your MCP │
      │ Server 1 │      │ Server 2 │
      └──────────┘      └──────────┘
```

## MCP Concepts

### Resources

Data that the AI model can read. Like GET endpoints:

```typescript
// Expose data the model can read
server.resource("experiments", "experiment://list", async () => ({
  contents: [{ text: JSON.stringify(experiments) }],
}));
```

### Tools

Functions the AI model can call. Like POST endpoints:

```typescript
// Expose functions the model can call
server.tool("run_experiment", { name: z.string() }, async ({ name }) => ({
  content: [{ type: "text", text: `Started experiment: ${name}` }],
}));
```

### Prompts

Reusable prompt templates:

```typescript
server.prompt("analyze", { topic: z.string() }, ({ topic }) => ({
  messages: [
    { role: "user", content: { type: "text", text: `Analyze ${topic} in detail.` } },
  ],
}));
```

## Building an MCP Server

### Installation

```bash
npm install @modelcontextprotocol/sdk zod
```

### Minimal Server

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "my-server",
  version: "1.0.0",
});

// Add a tool
server.tool(
  "greet",
  "Say hello to someone",
  { name: z.string() },
  async ({ name }) => ({
    content: [{ type: "text", text: `Hello, ${name}!` }],
  }),
);

// Start the server
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Transport

MCP servers communicate over **stdio** (standard input/output). The client launches the server as a child process and communicates via stdin/stdout using JSON-RPC.

This is why MCP servers log to stderr, not stdout:
```typescript
console.error("Debug info goes to stderr");  // visible in logs
console.log("This goes to stdout");           // this would interfere with MCP protocol!
```

### Tool Parameters with Zod

Tool parameters use Zod schemas — the same validation library from [Module 08](../08-web-servers-and-apis/README.md):

```typescript
server.tool(
  "search_experiments",
  "Search experiments by criteria",
  {
    query: z.string().optional().describe("Search term"),
    status: z.enum(["draft", "running", "completed", "failed"]).optional(),
    minAccuracy: z.number().min(0).max(1).optional(),
  },
  async ({ query, status, minAccuracy }) => {
    // Filter experiments based on parameters
    let results = experiments;
    if (query) results = results.filter(e => e.name.includes(query));
    if (status) results = results.filter(e => e.status === status);
    if (minAccuracy) results = results.filter(e => (e.accuracy ?? 0) >= minAccuracy);

    return {
      content: [{ type: "text", text: JSON.stringify(results, null, 2) }],
    };
  },
);
```

## Testing MCP Servers

### With the MCP Inspector

The SDK includes an inspector tool for testing:

```bash
npx @modelcontextprotocol/inspector npx tsx src/server.ts
```

This opens a web UI where you can:
- See all available tools and resources
- Call tools with custom parameters
- View responses

### With Claude Desktop

Add to your Claude Desktop config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["tsx", "/path/to/your/src/server.ts"]
    }
  }
}
```

Restart Claude Desktop and your tools will be available in the chat.

## Python Comparison

If you've used the Python MCP SDK, the TypeScript version is similar:

```python
# Python MCP
from mcp.server import Server
import mcp.types as types

server = Server("my-server")

@server.list_tools()
async def list_tools():
    return [types.Tool(name="greet", ...)]

@server.call_tool()
async def call_tool(name, arguments):
    if name == "greet":
        return [types.TextContent(text=f"Hello, {arguments['name']}!")]
```

```typescript
// TypeScript MCP
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.tool("greet", { name: z.string() }, async ({ name }) => ({
  content: [{ type: "text", text: `Hello, ${name}!` }],
}));
```

The TypeScript version uses a more declarative API — you define tools with `server.tool()` instead of implementing list/call handlers separately.

## Further Reading

- [MCP Specification](https://modelcontextprotocol.io)
- [TypeScript SDK Reference](https://github.com/modelcontextprotocol/typescript-sdk)
- [Example MCP Servers](https://github.com/modelcontextprotocol/servers)

## Exercises

1. [Basic MCP Server](./exercises/01-basic-mcp-server.md) — Build a server with resources and tools
2. [MCP with Tools](./exercises/02-mcp-with-tools.md) — Build an experiment tracker MCP server

---

**Next:** [Exercise 1: Basic MCP Server →](/11-mcp-servers/exercises/01-basic-mcp-server.md)
