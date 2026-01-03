# Chapter 3 Practice Files

This directory contains practice code for Chapter 3: MCP Concept and Tool/Resource Design.

## Directory Structure

```
chapter3/
├── code/               # MCP server implementations
├── data/output/        # Output files from MCP tools
└── docs/               # Design documents
```

## Running the Examples

### Minimal MCP Server

```bash
cd practice/chapter3
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
python3 code/3-2-minimal-server.py
```

### Tool Validation Example

```bash
python3 code/3-3-tool-validation.py
```

### Resource Server Example

```bash
python3 code/3-4-resource-server.py
```

### Complete MCP Server

```bash
python3 code/3-5-minimal-mcp-server.py
```

## Testing MCP Servers

MCP servers communicate via stdin/stdout. You can test them manually:

1. Run the server: `python3 code/3-5-minimal-mcp-server.py`
2. Send JSON-RPC request: `{"jsonrpc":"2.0","method":"tools/list","id":1}`
3. Press Enter to see the response

For automated testing, use the MCP client library or Claude Desktop.

## Requirements

- Python 3.9+
- No external dependencies (uses standard library only)
