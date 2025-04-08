# Claude Code MCP

[![smithery badge](https://smithery.ai/badge/@auchenberg/claude-code-mcp)](https://smithery.ai/server/@auchenberg/claude-code-mcp)

Claude Code MCP is an implementation of [Claude Code](https://gist.github.com/transitive-bullshit/487c9cb52c75a9701d312334ed53b20c) as a [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server. This project allows you to use Claude Code's powerful software engineering capabilities through the standardized MCP interface.

# ⚠️ DISCLAIMER ⚠️
Claude Code MCP is a auto-generated project by DevinAI, who was prompted to analyse the Claude Code codebase, and generate an MCP server.

This is a proof of concept that I don't advise anyone to use.

## What is Claude Code?

Claude Code is Anthropic's CLI tool for software engineering tasks, powered by Claude. It provides a set of tools and capabilities that help developers with:

- Code generation and editing
- Code review and analysis
- Debugging and troubleshooting
- File system operations
- Shell command execution
- Project exploration and understanding

The original implementation is available as a JavaScript module that defines prompts and tools for interacting with Claude's API.

## What is MCP?

The Model Context Protocol (MCP) is a standardized interface for AI models that enables consistent interaction patterns across different models and providers. MCP defines:

- **Tools**: Functions that models can call to perform actions
- **Resources**: External data that models can access
- **Prompts**: Predefined conversation templates

By implementing Claude Code as an MCP server, we make its capabilities available to any MCP-compatible client, allowing for greater interoperability and flexibility.

## Features

- Full implementation of Claude Code functionality as an MCP server
- Provides tools for file operations, shell commands, and code analysis
- Exposes resources for accessing file system and environment information
- Includes prompts for general CLI interaction and code review
- Compatible with any MCP client
- TypeScript implementation with full type safety

## Installation

### Installing via Smithery

To install Claude Code MCP Server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@auchenberg/claude-code-mcp):

```bash
npx -y @smithery/cli install @auchenberg/claude-code-mcp --client claude
```

### Installing Manually
```bash
# Clone the repository
git clone https://github.com/auchenberg/claude-code-mcp.git
cd claude-code-mcp

# Install dependencies
npm install

# Build the project
npm run build
```

## Usage

### Running as a standalone server

```bash
# Start the server
npm start
```

### Using with MCP clients

Claude Code MCP can be used with any MCP client. Here's an example of how to connect to it using the MCP TypeScript SDK:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

const transport = new StdioClientTransport({
  command: "node",
  args: ["dist/index.js"]
});

const client = new Client(
  {
    name: "example-client",
    version: "1.0.0"
  },
  {
    capabilities: {
      prompts: {},
      resources: {},
      tools: {}
    }
  }
);

await client.connect(transport);

// Use Claude Code through MCP
const result = await client.callTool({
  name: "bash",
  arguments: {
    command: "ls -la"
  }
});

console.log(result);
```

## Available Tools

Claude Code MCP provides the following tools:

- **bash**: Execute shell commands with security restrictions and timeout options
- **readFile**: Read files from the filesystem with options for line offsets and limits
- **listFiles**: List files and directories with detailed metadata
- **searchGlob**: Search for files matching a glob pattern
- **grep**: Search for text in files with regex pattern support
- **think**: A no-op tool for thinking through complex problems
- **codeReview**: Analyze and review code for bugs, security issues, and best practices
- **editFile**: Create or edit files with specified content

### Tool Details

#### bash

```typescript
{
  command: string;  // The shell command to execute
  timeout?: number; // Optional timeout in milliseconds (max 600000)
}
```

The bash tool includes security restrictions that prevent execution of potentially dangerous commands like `curl`, `wget`, and others.

#### readFile

```typescript
{
  file_path: string; // The absolute path to the file to read
  offset?: number;   // The line number to start reading from
  limit?: number;    // The number of lines to read
}
```

#### searchGlob

```typescript
{
  pattern: string;  // The glob pattern to match files against
  path?: string;    // The directory to search in (defaults to current working directory)
}
```

#### grep

```typescript
{
  pattern: string;   // The regular expression pattern to search for
  path?: string;     // The directory to search in (defaults to current working directory)
  include?: string;  // File pattern to include in the search (e.g. "*.js", "*.{ts,tsx}")
}
```

## Available Resources

- **file**: Access file contents (`file://{path}`)
  - Provides direct access to file contents with proper error handling
  - Returns the full text content of the specified file

- **directory**: List directory contents (`dir://{path}`)
  - Returns a JSON array of file information objects
  - Each object includes name, path, isDirectory, size, and modified date

- **environment**: Get system environment information (`env://info`)
  - Returns information about the system environment
  - Includes Node.js version, npm version, OS info, and environment variables

## Available Prompts

- **generalCLI**: General CLI prompt for Claude Code
  - Provides a comprehensive system prompt for Claude to act as a CLI tool
  - Includes guidelines for tone, style, proactiveness, and following conventions
  - Automatically includes environment details

- **codeReview**: Prompt for reviewing code
  - Specialized prompt for code review tasks
  - Analyzes code for bugs, security vulnerabilities, performance issues, and best practices

- **prReview**: Prompt for reviewing pull requests
  - Specialized prompt for PR review tasks
  - Analyzes PR changes and provides comprehensive feedback

- **initCodebase**: Initialize a new CLAUDE.md file with codebase documentation
  - Creates documentation for build/lint/test commands and code style guidelines
  - Useful for setting up a new project with Claude Code

## Development

```bash
# Run in development mode with auto-reload
npm run dev
```

## Architecture

Claude Code MCP is built with a modular architecture:

```
claude-code-mcp/
├── src/
│   ├── server/
│   │   ├── claude-code-server.ts  # Main server setup
│   │   ├── tools.ts               # Tool implementations
│   │   ├── prompts.ts             # Prompt definitions
│   │   └── resources.ts           # Resource implementations
│   ├── utils/
│   │   ├── bash.ts                # Shell command utilities
│   │   └── file.ts                # File system utilities
│   └── index.ts                   # Entry point
├── package.json
├── tsconfig.json
└── README.md
```

The implementation follows these key principles:

1. **Modularity**: Each component (tools, prompts, resources) is implemented in a separate module
2. **Type Safety**: Full TypeScript type definitions for all components
3. **Error Handling**: Comprehensive error handling for all operations
4. **Security**: Security restrictions for potentially dangerous operations

## Implementation Details

### MCP Server Setup

The main server is set up in `claude-code-server.ts`:

```typescript
export async function setupClaudeCodeServer(server: McpServer): Promise<void> {
  // Set up Claude Code tools
  setupTools(server);
  
  // Set up Claude Code prompts
  setupPrompts(server);
  
  // Set up Claude Code resources
  setupResources(server);
}
```

### Tool Implementation

Tools are implemented using the MCP SDK's tool registration method:

```typescript
server.tool(
  "toolName",
  "Tool description",
  {
    // Zod schema for tool arguments
    param1: z.string().describe("Parameter description"),
    param2: z.number().optional().describe("Optional parameter description")
  },
  async ({ param1, param2 }) => {
    // Tool implementation
    return {
      content: [{ type: "text", text: "Result" }]
    };
  }
);
```

### Resource Implementation

Resources are implemented using the MCP SDK's resource registration method:

```typescript
server.resource(
  "resourceName",
  new ResourceTemplate("resource://{variable}", { list: undefined }),
  async (uri, variables) => {
    // Resource implementation
    return {
      contents: [{
        uri: uri.href,
        text: "Resource content"
      }]
    };
  }
);
```

## License

MIT

## Acknowledgements

- [Claude Code](https://gist.github.com/transitive-bullshit/487c9cb52c75a9701d312334ed53b20c) by Anthropic
- [Model Context Protocol](https://modelcontextprotocol.io)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Disclaimer

This project is not officially affiliated with Anthropic. Claude Code is a product of Anthropic, and this project is an independent implementation of Claude Code as an MCP server.
