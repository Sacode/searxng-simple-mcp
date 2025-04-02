# SearxNG MCP Server

A Model Context Protocol (MCP) server that provides web search capabilities using SearxNG, allowing AI assistants like Claude to search the web.

> *Created by AI with human supervision - because sometimes even artificial intelligence needs someone to tell it when to take a coffee break! 🤖☕*

## Overview

This project implements an MCP server that connects to SearxNG, a privacy-respecting metasearch engine. The server provides a simple and efficient way for Large Language Models to search the web without tracking users.

The server is specifically designed for LLMs and includes only essential features to minimize context window usage. This streamlined approach ensures efficient communication between LLMs and the search engine, preserving valuable context space for more important information.

### Features

- Privacy-focused web search through SearxNG
- Simple API for LLM integration
- Compatible with Claude Desktop and other MCP-compliant clients
- Configurable search parameters
- Clean, formatted search results optimized for LLMs

## Integration with MCP-Compatible Applications

### Integration Examples

#### Using pipx run (Recommended, no installation required)

Create a `.clauderc` file in your home directory:

```json
{
  "mcpServers": {
    "searxng": {
      "command": "pipx",
      "args": [
        "run", "searxng-simple-mcp"
      ],
      "transport": "stdio",
      "env": {
        "SEARXNG_MCP_SEARXNG_URL": "https://your-instance.example.com"
      }
    }
  }
}
```

#### Using uvx run (No installation required)

```json
{
  "mcpServers": {
    "searxng": {
      "command": "uvx",
      "args": [
        "run", "searxng-simple-mcp"
      ],
      "transport": "stdio",
      "env": {
        "SEARXNG_MCP_SEARXNG_URL": "https://your-instance.example.com"
      }
    }
  }
}
```

#### Using Python with pip (requires installation)

```json
{
  "mcpServers": {
    "searxng": {
      "command": "python",
      "args": ["-m", "searxng_simple_mcp.server"],
      "transport": "stdio",
      "env": {
        "SEARXNG_MCP_SEARXNG_URL": "https://your-instance.example.com"
      }
    }
  }
}
```

#### Using with Docker (No installation required)

```json
{
  "mcpServers": {
    "searxng": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i", "--network=host",
        "-e", "TRANSPORT_PROTOCOL=stdio",
        "-e", "SEARXNG_MCP_SEARXNG_URL=http://localhost:8080",
        "ghcr.io/sacode/searxng-simple-mcp:latest"
      ],
      "transport": "stdio"
    }
  }
}
```

**Note:** When using Docker with MCP servers:

1. Environment variables must be passed directly using the `-e` flag in the `args` array, as the `env` object is not properly passed to the Docker container.
2. If you need to access a SearxNG instance running on localhost (e.g., <http://localhost:8080>), you must use the `--network=host` flag to allow the container to access the host's network. Otherwise, "localhost" inside the container will refer to the container itself, not your host machine.
3. When using `--network=host`, port mappings (`-p`) are not needed and will be ignored, as the container shares the host's network stack directly.

## Configuration

Configure the server using environment variables:

| Environment Variable | Description | Default Value |
|----------------------|-------------|---------------|
| SEARXNG_MCP_SEARXNG_URL | URL of the SearxNG instance to use | <https://paulgo.io/> |
| SEARXNG_MCP_TIMEOUT | HTTP request timeout in seconds | 10 |
| SEARXNG_MCP_DEFAULT_RESULT_COUNT | Default number of results to return | 10 |
| SEARXNG_MCP_DEFAULT_LANGUAGE | Language code for results (e.g., 'en', 'ru', 'all') | all |
| SEARXNG_MCP_DEFAULT_FORMAT | Default format for results ('text', 'json') | text |
| TRANSPORT_PROTOCOL | Transport protocol ('stdio' or 'sse') | stdio |

You can find a list of public SearxNG instances at [https://searx.space](https://searx.space) if you don't want to host your own.

## Installation & Usage

### Prerequisites

- Python 3.10 or higher
- A SearxNG instance (public or self-hosted)

### Option 1: Run Without Installation (Recommended)

The easiest way to use this server is with pipx or uvx, which allows you to run the package without installing it permanently:

```bash
# Using pipx
pip install pipx  # Install pipx if you don't have it
pipx run searxng-simple-mcp

# OR using uvx
pip install uvx  # Install uvx if you don't have it
uvx run searxng-simple-mcp
```

You can pass configuration options directly:

```bash
# Using pipx with custom SearxNG instance
pipx run searxng-simple-mcp --searxng-url https://your-instance.example.com
```

### Option 2: Install from PyPI or Source

For more permanent installation:

```bash
# From PyPI using pip
pip install searxng-simple-mcp

# OR using uv (faster installation)
pip install uv
uv pip install searxng-simple-mcp

# OR from source
git clone https://github.com/Sacode/searxng-simple-mcp.git
cd searxng-simple-mcp
pip install uv
uv pip install -e .
```

After installation, you can run the server with:

```bash
# Run directly after installation
python -m searxng_simple_mcp.server

# OR with configuration options
python -m searxng_simple_mcp.server --searxng-url https://your-instance.example.com
```

### Option 3: Docker

If you prefer using Docker:

```bash
# Pull the Docker image
docker pull ghcr.io/sacode/searxng-simple-mcp:latest

# Run the container
docker run -p 8000:8000 --env-file .env ghcr.io/sacode/searxng-simple-mcp:latest
```

## Transport Protocols

The MCP server supports two transport protocols:

- **STDIO**: For CLI applications and direct integration (default)
- **SSE**: For web-based clients and HTTP-based integrations

### Using SSE with Docker

Server-Sent Events (SSE) is a transport protocol that allows the server to push updates to clients over HTTP connections. This is useful for web-based applications and services that need real-time updates from the MCP server.

To use SSE with Docker:

```bash
# Run with SSE transport protocol
docker run -p 8000:8000 -e TRANSPORT_PROTOCOL=sse -e SEARXNG_MCP_SEARXNG_URL=https://your-instance.example.com ghcr.io/sacode/searxng-simple-mcp:latest
```

When using SSE, the server will be accessible via HTTP at `http://localhost:8000` by default.

You can find a complete example in the `docker-compose.yml` file included in this repository:

```yaml
environment:
  - SEARXNG_MCP_SEARXNG_URL=https://searx.info
  - SEARXNG_MCP_TIMEOUT=10
  - SEARXNG_MCP_MAX_RESULTS=20
  - SEARXNG_MCP_LANGUAGE=all
  - TRANSPORT_PROTOCOL=sse # Transport protocol: stdio or sse
```

**Note:** Not all applications support the SSE transport protocol. Make sure your MCP client is compatible with SSE before using this transport method. Some applications may only support the STDIO transport protocol.

To connect to the SSE server from an MCP client, you would use a configuration like:

```json
{
  "mcpServers": {
    "searxng": {
      "url": "http://localhost:8000",
      "transport": "sse"
    }
  }
}
```

## Development

For development and testing:

```bash
# Install dependencies
uv pip install -e .

# Run linter and formatter
ruff check .
ruff check --fix .
ruff format .

# Run the server directly
python -m src.searxng_simple_mcp.server

# OR using FastMCP
fastmcp run src/searxng_simple_mcp/server.py  # Use stdio transport (default)
fastmcp run src/searxng_simple_mcp/server.py --transport sse  # Use sse transport

# Run in development mode (launches MCP Inspector)
fastmcp dev src/searxng_simple_mcp/server.py
```

## Docker Usage

Docker is an alternative way to run the server:

```bash
# Using pre-built image
docker pull ghcr.io/sacode/searxng-simple-mcp:latest
docker run -p 8000:8000 --env-file .env ghcr.io/sacode/searxng-simple-mcp:latest

# Building locally
docker build -t searxng-simple-mcp:local .
docker run -p 8000:8000 --env-file .env searxng-simple-mcp:local

# Using Docker Compose
docker-compose up -d
```

## Package Structure

```
searxng-simple-mcp/
├── src/
│   ├── run_server.py         # Entry point script
│   └── searxng_simple_mcp/   # Main package
├── docker-compose.yml        # Docker Compose configuration
├── Dockerfile                # Docker configuration
└── pyproject.toml            # Python project configuration
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.
