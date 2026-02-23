# Exercise: Basic MCP Server

Build your first MCP server with tools and resources.

## Setup

```bash
mkdir mcp-basics && cd mcp-basics
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D typescript tsx
```

Add to `package.json`: `"type": "module"`

## Tasks

### 1. Create a minimal server

Create `src/server.ts` with:

- Server name: `"learning-tools"`
- A `greet` tool that takes a `name` parameter and returns a greeting
- Connect via `StdioServerTransport`

Test it with the MCP Inspector:
```bash
npx @modelcontextprotocol/inspector npx tsx src/server.ts
```

### 2. Add a `calculate` tool

Add a tool that evaluates math expressions:
- Parameter: `expression` (string)
- Returns the result as text
- Handle errors (invalid expressions) gracefully

### 3. Add a `convert_units` tool

Add a tool for unit conversion:
- Parameters: `value` (number), `from` (string), `to` (string)
- Support at least: km↔miles, kg↔lbs, celsius↔fahrenheit
- Return a formatted result like "5 km = 3.11 miles"

### 4. Add a resource

Add a resource that exposes a list of supported conversions:
- URI: `conversions://supported`
- Returns a JSON list of available conversion pairs

### 5. Add a prompt template

Add a prompt called `unit-helper` that generates a system prompt for a unit conversion assistant. It should take an optional `specialty` parameter (e.g., "cooking", "science") to customize the prompt.

<details>
<summary>Hint: Server structure</summary>

```typescript
import { McpServer, ResourceTemplate } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "learning-tools",
  version: "1.0.0",
});

// Tools
server.tool("name", "description", { param: z.string() }, async ({ param }) => ({
  content: [{ type: "text", text: "result" }],
}));

// Resources
server.resource("name", "uri://path", async () => ({
  contents: [{ uri: "uri://path", text: "data" }],
}));

// Prompts
server.prompt("name", "description", { param: z.string().optional() }, ({ param }) => ({
  messages: [
    { role: "user", content: { type: "text", text: "prompt text" } },
  ],
}));

const transport = new StdioServerTransport();
await server.connect(transport);
```

Remember: use `console.error()` for logging, not `console.log()`.

</details>

<details>
<summary>Solution</summary>

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "learning-tools",
  version: "1.0.0",
});

// --- Tool: greet ---

server.tool(
  "greet",
  "Say hello to someone",
  { name: z.string().describe("Name of the person to greet") },
  async ({ name }) => ({
    content: [{ type: "text", text: `Hello, ${name}! Welcome to MCP.` }],
  }),
);

// --- Tool: calculate ---

server.tool(
  "calculate",
  "Evaluate a mathematical expression",
  {
    expression: z.string().describe("Math expression, e.g. '2 + 3 * 4' or 'Math.sqrt(16)'"),
  },
  async ({ expression }) => {
    try {
      const result = new Function(`return ${expression}`)();
      return {
        content: [{ type: "text", text: `${expression} = ${result}` }],
      };
    } catch {
      return {
        content: [{ type: "text", text: `Error: could not evaluate "${expression}"` }],
        isError: true,
      };
    }
  },
);

// --- Tool: convert_units ---

const conversions: Record<string, Record<string, (v: number) => number>> = {
  km: { miles: (v) => v * 0.621371 },
  miles: { km: (v) => v * 1.60934 },
  kg: { lbs: (v) => v * 2.20462 },
  lbs: { kg: (v) => v * 0.453592 },
  celsius: { fahrenheit: (v) => v * 9 / 5 + 32 },
  fahrenheit: { celsius: (v) => (v - 32) * 5 / 9 },
  meters: { feet: (v) => v * 3.28084 },
  feet: { meters: (v) => v * 0.3048 },
  liters: { gallons: (v) => v * 0.264172 },
  gallons: { liters: (v) => v * 3.78541 },
};

server.tool(
  "convert_units",
  "Convert between units of measurement",
  {
    value: z.number().describe("The value to convert"),
    from: z.string().describe("Source unit (e.g. 'km', 'celsius', 'kg')"),
    to: z.string().describe("Target unit (e.g. 'miles', 'fahrenheit', 'lbs')"),
  },
  async ({ value, from, to }) => {
    const fromLower = from.toLowerCase();
    const toLower = to.toLowerCase();

    const converter = conversions[fromLower]?.[toLower];
    if (!converter) {
      const supported = Object.entries(conversions)
        .flatMap(([from, targets]) => Object.keys(targets).map((to) => `${from} → ${to}`))
        .join(", ");
      return {
        content: [{ type: "text", text: `Unsupported conversion: ${from} → ${to}. Supported: ${supported}` }],
        isError: true,
      };
    }

    const result = converter(value);
    return {
      content: [{ type: "text", text: `${value} ${fromLower} = ${result.toFixed(4)} ${toLower}` }],
    };
  },
);

// --- Resource: supported conversions ---

server.resource(
  "supported-conversions",
  "conversions://supported",
  async () => {
    const pairs = Object.entries(conversions).flatMap(([from, targets]) =>
      Object.keys(targets).map((to) => ({ from, to }))
    );

    return {
      contents: [{
        uri: "conversions://supported",
        text: JSON.stringify(pairs, null, 2),
      }],
    };
  },
);

// --- Prompt: unit-helper ---

server.prompt(
  "unit-helper",
  "Generate a system prompt for a unit conversion assistant",
  { specialty: z.string().optional().describe("Optional specialty like 'cooking' or 'science'") },
  ({ specialty }) => {
    const base = "You are a helpful unit conversion assistant. You can convert between various units of measurement.";
    const specialtyText = specialty
      ? ` You specialize in ${specialty}-related conversions and can provide context about why certain units are used in ${specialty}.`
      : "";

    return {
      messages: [
        {
          role: "user" as const,
          content: {
            type: "text" as const,
            text: base + specialtyText + " When converting, always show your work and explain the conversion factor.",
          },
        },
      ],
    };
  },
);

// --- Start server ---

console.error("Starting learning-tools MCP server...");
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Testing

```bash
# Open the MCP Inspector
npx @modelcontextprotocol/inspector npx tsx src/server.ts
```

In the inspector:
1. Click "Tools" to see your three tools
2. Call `greet` with `{ "name": "Alice" }`
3. Call `calculate` with `{ "expression": "2 ** 10" }`
4. Call `convert_units` with `{ "value": 100, "from": "celsius", "to": "fahrenheit" }`
5. Click "Resources" to see the supported conversions
6. Click "Prompts" to see the unit-helper prompt

### To use with Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "learning-tools": {
      "command": "npx",
      "args": ["tsx", "/absolute/path/to/mcp-basics/src/server.ts"]
    }
  }
}
```

Then restart Claude Desktop. You'll see the tools available in the chat.

</details>

---

**Next:** [Exercise 2: MCP with Tools →](/11-mcp-servers/exercises/02-mcp-with-tools.md)
