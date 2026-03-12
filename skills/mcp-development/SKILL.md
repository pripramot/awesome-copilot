---
name: mcp-development
description: 'Develop Model Context Protocol (MCP) servers and clients to extend AI assistant capabilities with custom tools, resources, and prompts. Covers server creation in TypeScript and Python, tool definitions, resource providers, and integration with GitHub Copilot and Claude.'
---

# MCP Development (พัฒนา Model Context Protocol)

Build MCP servers that extend AI assistants with custom tools, resources, and capabilities.

## What is MCP?

Model Context Protocol (MCP) is an open standard that enables AI assistants to securely connect with external data sources and tools. MCP servers provide:
- **Tools** - Functions the AI can call (e.g., search database, run code)
- **Resources** - Data the AI can read (e.g., files, API responses)
- **Prompts** - Reusable prompt templates

## TypeScript MCP Server

```bash
# Create new MCP server
npm create @modelcontextprotocol/server my-mcp-server
cd my-mcp-server
npm install
```

```typescript
// src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  {
    name: "my-custom-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
      resources: {},
    },
  }
);

// Define available tools
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "search_products",
        description: "Search for products in the database",
        inputSchema: {
          type: "object",
          properties: {
            query: {
              type: "string",
              description: "Search query",
            },
            category: {
              type: "string",
              enum: ["electronics", "clothing", "food"],
              description: "Product category to filter by",
            },
          },
          required: ["query"],
        },
      },
    ],
  };
});

// Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "search_products") {
    const { query, category } = args as { query: string; category?: string };

    // Your business logic here
    const results = await searchDatabase(query, category);

    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(results, null, 2),
        },
      ],
    };
  }

  throw new Error(`Unknown tool: ${name}`);
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

## Python MCP Server

```python
# server.py
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp import types

server = Server("thai-business-tools")

@server.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="validate_thai_id",
            description="Validate a Thai National ID card number",
            inputSchema={
                "type": "object",
                "properties": {
                    "id_number": {
                        "type": "string",
                        "description": "13-digit Thai National ID number"
                    }
                },
                "required": ["id_number"]
            }
        ),
        types.Tool(
            name="format_thai_date",
            description="Convert a date to Thai Buddhist Era format",
            inputSchema={
                "type": "object",
                "properties": {
                    "date": {
                        "type": "string",
                        "description": "Date in ISO format (YYYY-MM-DD)"
                    }
                },
                "required": ["date"]
            }
        )
    ]

@server.call_tool()
async def call_tool(
    name: str, arguments: dict
) -> list[types.TextContent]:
    if name == "validate_thai_id":
        id_number = arguments["id_number"]
        is_valid = validate_thai_national_id(id_number)
        return [types.TextContent(
            type="text",
            text=f"ID {id_number} is {'valid' if is_valid else 'invalid'}"
        )]

    if name == "format_thai_date":
        from datetime import datetime
        dt = datetime.fromisoformat(arguments["date"])
        thai_year = dt.year + 543
        thai_months = [
            "มกราคม", "กุมภาพันธ์", "มีนาคม", "เมษายน",
            "พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม",
            "กันยายน", "ตุลาคม", "พฤศจิกายน", "ธันวาคม"
        ]
        result = f"{dt.day} {thai_months[dt.month-1]} พ.ศ. {thai_year}"
        return [types.TextContent(type="text", text=result)]

    raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream, write_stream,
            server.create_initialization_options()
        )

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

## MCP Configuration for GitHub Copilot

```json
// .github/copilot/mcp.json or ~/.config/github-copilot/mcp.json
{
  "servers": {
    "my-mcp-server": {
      "type": "stdio",
      "command": "node",
      "args": ["dist/index.js"],
      "cwd": "/path/to/my-mcp-server"
    },
    "thai-business-tools": {
      "type": "stdio",
      "command": "python",
      "args": ["server.py"],
      "cwd": "/path/to/thai-business-tools"
    },
    "docker-mcp": {
      "type": "stdio",
      "command": "docker",
      "args": ["run", "-i", "--rm", "my-mcp-image:latest"]
    }
  }
}
```

## Adding Resources

```typescript
import { ListResourcesRequestSchema, ReadResourceRequestSchema } from "@modelcontextprotocol/sdk/types.js";

// List available resources
server.setRequestHandler(ListResourcesRequestSchema, async () => ({
  resources: [
    {
      uri: "file:///config/settings.json",
      name: "Application Settings",
      mimeType: "application/json",
    },
  ],
}));

// Read a resource
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  if (request.params.uri === "file:///config/settings.json") {
    const content = await fs.readFile("./config/settings.json", "utf-8");
    return {
      contents: [{ uri: request.params.uri, text: content, mimeType: "application/json" }],
    };
  }
  throw new Error(`Unknown resource: ${request.params.uri}`);
});
```

## Best Practices

1. **Descriptive tool names** - Use clear, verb-noun names like `search_products`, `create_user`
2. **JSON Schema validation** - Define complete input schemas with descriptions
3. **Error handling** - Return meaningful error messages, don't expose internal details
4. **Stateless design** - Design tools to be stateless where possible for reliability
5. **Docker packaging** - Package servers as Docker images for easy distribution
6. **Test with MCP Inspector** - Use `npx @modelcontextprotocol/inspector` to debug
