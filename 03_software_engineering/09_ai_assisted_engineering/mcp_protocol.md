---
layer: 03_software_engineering
type: engineering
tool: MCP
status: growing
tags: [mcp, model-context-protocol, llm, tools, ai-infrastructure]
created: 2026-03-05
---

# MCP Protocol

## Purpose

The Model Context Protocol (MCP) is an open standard, introduced by Anthropic in November 2024, that defines how LLM-based applications connect to external tools, data sources, and services. MCP solves the N×M integration problem: instead of each LLM host building custom integrations with each data source, both sides implement a single protocol. The result is a composable ecosystem where any MCP-compliant host can use any MCP-compliant server.

## Architecture

### Components

```
┌───────────────────────────────────────────────────────────────┐
│                        MCP Host                               │
│  (Claude Desktop, Copilot, Cursor, custom LLM application)   │
│                                                               │
│   ┌─────────────────┐      ┌──────────────────────────────┐  │
│   │   LLM / Model   │◀────▶│         MCP Client           │  │
│   └─────────────────┘      │  (manages server connections) │  │
│                             └──────────────┬───────────────┘  │
└────────────────────────────────────────────┼──────────────────┘
                                             │ MCP Protocol
               ┌─────────────────────────────┼────────────────┐
               │                             │                │
               ▼                             ▼                ▼
      ┌─────────────────┐  ┌──────────────────┐  ┌──────────────────┐
      │   MCP Server    │  │   MCP Server     │  │   MCP Server     │
      │  (filesystem)   │  │   (postgres)     │  │   (github)       │
      └─────────────────┘  └──────────────────┘  └──────────────────┘
```

**Host** — the LLM application that initiates connections to MCP servers. Controls which servers are available and mediates tool calls between the model and servers.

**Client** — runs inside the host. Manages the lifecycle of connections to one or more MCP servers.

**Server** — a lightweight process that exposes capabilities (tools, resources, prompts) via the MCP protocol. Runs as a separate process, connecting via stdio or SSE.

### Three Primitive Types

| Primitive | Direction | Purpose |
|-----------|-----------|---------|
| **Tools** | Model → Server | Functions the model can invoke (side-effectful) |
| **Resources** | Server → Model | Data/context the model can read (read-only, URI-addressed) |
| **Prompts** | Server → Model | Reusable prompt templates with arguments |

### Transport Mechanisms

**stdio** — the server is launched as a subprocess; host communicates via stdin/stdout. Simplest, most common for local tools.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"],
      "env": {}
    }
  }
}
```

**SSE (Server-Sent Events)** — host connects to a running HTTP server. Used for remote MCP servers, persistent servers, or multi-client scenarios.

```json
{
  "mcpServers": {
    "remote-db": {
      "url": "https://mcp.internal.example.com/sse",
      "headers": { "Authorization": "Bearer ${DB_MCP_TOKEN}" }
    }
  }
}
```

### Protocol Flow

```
Host                          MCP Server
 │                                │
 │──── initialize ───────────────▶│  exchange capabilities
 │◀─── initialized ───────────────│
 │                                │
 │──── tools/list ───────────────▶│  discover available tools
 │◀─── tools/list result ─────────│
 │                                │
 │  [Model decides to call tool]  │
 │                                │
 │──── tools/call ───────────────▶│  invoke with arguments
 │◀─── tools/call result ─────────│  receive result
 │                                │
 │  [Result injected into context]│
```

Messages are JSON-RPC 2.0. Each tool call is a separate request/response cycle.

## Implementation Notes

### Tool Calling via MCP

From the model's perspective, tools are just function signatures in the system context:

```json
{
  "name": "read_file",
  "description": "Read the complete contents of a file from the filesystem.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "path": {
        "type": "string",
        "description": "The absolute path of the file to read."
      }
    },
    "required": ["path"]
  }
}
```

The model generates a tool use block; the MCP client serialises it, calls the server, and returns the result as a tool result message.

### Building an MCP Server — Python SDK

```bash
pip install mcp
```

**Minimal server with tools and resources:**

```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent, Resource
import asyncio
import json
from pathlib import Path

app = Server("my-data-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="query_database",
            description="Run a read-only SQL query against the analytics database.",
            inputSchema={
                "type": "object",
                "properties": {
                    "sql": {
                        "type": "string",
                        "description": "SELECT query to execute. Must be read-only."
                    },
                    "limit": {
                        "type": "integer",
                        "description": "Maximum rows to return. Default 100.",
                        "default": 100
                    }
                },
                "required": ["sql"]
            }
        ),
        Tool(
            name="list_tables",
            description="List all tables available in the database.",
            inputSchema={"type": "object", "properties": {}}
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "query_database":
        sql = arguments["sql"]
        limit = arguments.get("limit", 100)
        # Validate: only allow SELECT
        if not sql.strip().upper().startswith("SELECT"):
            return [TextContent(type="text", text="Error: only SELECT queries are permitted.")]
        results = await run_query(sql, limit)
        return [TextContent(type="text", text=json.dumps(results, indent=2))]

    elif name == "list_tables":
        tables = await get_tables()
        return [TextContent(type="text", text="\n".join(tables))]

    return [TextContent(type="text", text=f"Unknown tool: {name}")]

@app.list_resources()
async def list_resources() -> list[Resource]:
    return [
        Resource(
            uri="db://schema/full",
            name="Database Schema",
            description="Complete schema for all tables.",
            mimeType="application/json"
        )
    ]

@app.read_resource()
async def read_resource(uri: str) -> str:
    if uri == "db://schema/full":
        schema = await get_schema()
        return json.dumps(schema, indent=2)
    raise ValueError(f"Unknown resource: {uri}")

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

**TypeScript / Node.js SDK:**

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { CallToolRequestSchema, ListToolsRequestSchema } from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "my-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: "fetch_url",
    description: "Fetch content from a URL.",
    inputSchema: {
      type: "object",
      properties: { url: { type: "string" } },
      required: ["url"]
    }
  }]
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "fetch_url") {
    const { url } = request.params.arguments as { url: string };
    const response = await fetch(url);
    const text = await response.text();
    return { content: [{ type: "text", text }] };
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### Official MCP Server Directory

The [MCP servers repository](https://github.com/modelcontextprotocol/servers) provides reference implementations:

| Server | Transport | Exposes |
|--------|-----------|---------|
| `filesystem` | stdio | File read/write/list/search |
| `github` | stdio | Repos, issues, PRs, commits |
| `postgres` | stdio | DB query, schema inspection |
| `sqlite` | stdio | Local SQLite query |
| `brave-search` | stdio | Web search via Brave API |
| `fetch` | stdio | HTTP GET with HTML→Markdown |
| `memory` | stdio | Persistent key-value memory |
| `puppeteer` | stdio | Browser automation |
| `slack` | stdio | Send messages, read channels |

### Security Considerations

- MCP servers run as separate processes — they execute arbitrary code. Only connect to servers you trust.
- Servers should validate and sanitise all tool arguments before acting on them.
- Use the principle of least privilege: filesystem servers should be scoped to a specific directory, not `/`.
- For remote MCP servers, authenticate requests (Bearer token, mTLS).
- Tool descriptions are part of the attack surface — **prompt injection** can occur if tool results contain adversarial instructions that the model acts on.

### MCP in Claude Desktop

`~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/Documents"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..." }
    },
    "my-db": {
      "command": "python",
      "args": ["/path/to/my_db_server.py"],
      "env": { "DATABASE_URL": "postgresql://..." }
    }
  }
}
```

## Trade-offs

| Approach | Pro | Con |
|----------|-----|-----|
| stdio transport | Simple, no networking | Single host only |
| SSE transport | Multi-host, remote | Network overhead, auth required |
| Python SDK | Easy to write, rich ecosystem | Slower startup than Node |
| Node.js SDK | Fast startup | TypeScript typing required for best DX |
| Broad tool permissions | Powerful | Security risk; model may do unintended things |
| Narrow tool permissions | Safe | May need many servers |
| Remote MCP server | Centralised, shared | Latency, auth complexity |

## References

- [MCP specification](https://spec.modelcontextprotocol.io/)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Official MCP servers](https://github.com/modelcontextprotocol/servers)
- [Anthropic MCP announcement](https://www.anthropic.com/news/model-context-protocol)

## Links
- [[agentic_coding|Agentic Coding]]
- [[llm_code_generation|LLM Code Generation]]
- [[repo_rag_for_code|Repo RAG for Code]]
- [[05_ai_engineering/03_rag_and_agents/function_calling|Function Calling]]
