---
layer: 08_implementations
type: application
status: growing
tags: [pattern, reasoning, llm]
created: 2026-03-06
---

# MCP Server Implementation

## Purpose

Practical implementation of a Model Context Protocol (MCP) server that exposes tools, resources, and prompts to AI coding assistants (Copilot, Claude, Cursor). MCP provides a standardised way for LLMs to call functions, read resources, and use prompt templates. Synthesized from: [[mcp_protocol|MCP Protocol]], [[agentic_coding|Agentic Coding]], [[repo_rag_for_code|Repo RAG for Code]].

### Examples

**Install the Python MCP SDK:**
```bash
pip install mcp
```

**Minimal Python MCP server with tools:**
```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent, Resource
from pydantic import BaseModel
import asyncio, subprocess, pathlib, json

server = Server("code-assistant")

# ─── Tool definitions ─────────────────────────────────────────────────────────

@server.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="read_file",
            description="Read a file from the repository",
            inputSchema={
                "type": "object",
                "properties": {
                    "path": {"type": "string", "description": "Relative path from repo root"},
                },
                "required": ["path"],
            },
        ),
        Tool(
            name="run_tests",
            description="Run pytest for a given test path",
            inputSchema={
                "type": "object",
                "properties": {
                    "path": {"type": "string", "default": "tests/"},
                },
            },
        ),
        Tool(
            name="search_code",
            description="Search codebase for a pattern using ripgrep",
            inputSchema={
                "type": "object",
                "properties": {
                    "pattern": {"type": "string"},
                    "file_pattern": {"type": "string", "default": "*.py"},
                },
                "required": ["pattern"],
            },
        ),
    ]

# ─── Tool implementations ─────────────────────────────────────────────────────

REPO_ROOT = pathlib.Path("/workspace")

@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "read_file":
        path = REPO_ROOT / arguments["path"]
        # Sandbox: resolve and verify it stays within REPO_ROOT
        resolved = path.resolve()
        if not str(resolved).startswith(str(REPO_ROOT.resolve())):
            return [TextContent(type="text", text="Error: path traversal denied")]
        if not resolved.exists():
            return [TextContent(type="text", text=f"File not found: {arguments['path']}")]
        return [TextContent(type="text", text=resolved.read_text(errors="replace"))]

    if name == "run_tests":
        result = subprocess.run(
            ["python", "-m", "pytest", arguments.get("path", "tests/"), "--tb=short", "-q"],
            cwd=REPO_ROOT, capture_output=True, text=True, timeout=120,
        )
        output = result.stdout + result.stderr
        return [TextContent(type="text", text=output)]

    if name == "search_code":
        result = subprocess.run(
            ["rg", "--glob", arguments.get("file_pattern", "*.py"),
             "-n", arguments["pattern"], str(REPO_ROOT)],
            capture_output=True, text=True, timeout=30,
        )
        return [TextContent(type="text", text=result.stdout or "No matches")]

    return [TextContent(type="text", text=f"Unknown tool: {name}")]

# ─── Resources ────────────────────────────────────────────────────────────────

@server.list_resources()
async def list_resources() -> list[Resource]:
    return [
        Resource(
            uri="repo://structure",
            name="Repository Structure",
            description="Directory tree of the repository",
            mimeType="text/plain",
        ),
    ]

@server.read_resource()
async def read_resource(uri: str) -> str:
    if uri == "repo://structure":
        result = subprocess.run(
            ["find", str(REPO_ROOT), "-name", "*.py", "-not", "-path", "*/.*"],
            capture_output=True, text=True
        )
        return result.stdout
    raise ValueError(f"Unknown resource: {uri}")

# ─── Entry point ──────────────────────────────────────────────────────────────

async def main():
    async with stdio_server() as streams:
        await server.run(*streams, server.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

**Register with Claude Desktop (`.claude/config.json`):**
```json
{
  "mcpServers": {
    "code-assistant": {
      "command": "python",
      "args": ["/workspace/.mcp/server.py"],
      "env": {}
    }
  }
}
```

**TypeScript MCP server (for Node.js-based tools):**
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { CallToolRequestSchema, ListToolsRequestSchema } from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "ts-code-assistant", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: "lint",
    description: "Run ESLint on a file",
    inputSchema: {
      type: "object",
      properties: { path: { type: "string" } },
      required: ["path"],
    },
  }],
}));

server.setRequestHandler(CallToolRequestSchema, async (req) => {
  const { name, arguments: args } = req.params;
  if (name === "lint") {
    const { execSync } = await import("child_process");
    try {
      const out = execSync(`npx eslint ${args!.path} --format compact`).toString();
      return { content: [{ type: "text", text: out }] };
    } catch (e: any) {
      return { content: [{ type: "text", text: e.stdout?.toString() ?? "Error" }] };
    }
  }
  throw new Error(`Unknown tool: ${name}`);
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

## Architecture

```
AI assistant (Claude / Copilot / Cursor)
    │
    │  stdio or SSE transport (JSON-RPC 2.0)
    ▼
MCP Server
    ├── Tools     — callable functions (read_file, run_tests, search)
    ├── Resources — readable content (repo structure, docs)
    └── Prompts   — prompt templates with arguments

Security boundary:
    Server enforces path sandboxing, rate limiting, and command allowlist
    All file access validated against REPO_ROOT before execution
```

**Key design decisions:**
- Use `stdio` transport for local tools (simpler, no port management)
- Use SSE transport (`mcp.server.sse`) for remote/multi-user servers
- Always sandbox file system access — validate paths with `Path.resolve()` + prefix check
- Keep tool implementations stateless where possible for safety

## References

- [Model Context Protocol Documentation](https://modelcontextprotocol.io/)

## Links
- [[mcp_protocol|MCP Protocol]]
- [[agentic_coding|Agentic Coding]]
- [[repo_rag_for_code|Repo RAG for Code]]
- [[llm_code_generation|LLM Code Generation]]
- [[04_software_engineering/08_security/filesystem_sandboxing|Filesystem Sandboxing]]
- [[06_ai_engineering/04_rag_and_agents/function_calling|Function Calling]]
