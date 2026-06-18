# MCP Security Platform

A reference implementation of the adapter pattern over MCP (Model
Context Protocol). Wrap any REST or GraphQL API as tools that an LLM
agent can call through a uniform interface.

**Public repository:** https://github.com/Koslovski79/platform-adapter-mcp

## What it does

Exposes MCP servers (the standard tool-integration protocol for LLM
agents) as adapters over external security platforms. The same agent
loop that calls one tool can call any wrapped API, with auth, rate
limiting, and error handling normalized at the adapter layer.

This lets agents operate against platforms that were never designed
for programmatic LLM access — they get clean tool semantics, schema
validation, and async execution without each integration re-inventing
the same boilerplate.

## Architecture

```
  +-----------------+        +---------------------+
  |   LLM Agent     |  stdio |   MCP Server        |
  |  (OpenCode,     | <----> |                     |
  |   Claude Code,  |  JSON  |   +---------------+  |
  |   Hermes, ...)  |  RPC   |   |  Tool Registry|  |
  +-----------------+        |   +-------+-------+  |
                             |           |          |
                             |   +-------v-------+  |
                             |   |   Adapters    |  |
                             |   |  (REST/GraphQL|  |
                             |   |   /CLI/DB)    |  |
                             |   +---------------+  |
                             +---------------------+
```

## Stack

- **Python 3.11+**
- **MCP (Model Context Protocol)** — tool integration standard
- **asyncio** — concurrent tool execution
- **httpx / aiohttp** — async HTTP clients
- **Pydantic** — schema validation for tool inputs/outputs
- **Caido / Headroom / Kali tools** — example adapter targets

## Why it matters

MCP is becoming the standard way LLM agents talk to the outside world.
But most real-world APIs weren't built for that — they have auth quirks,
rate limits, weird schemas, paginated responses. Writing one adapter
per integration is repetitive.

This platform turns "wrap a REST endpoint as an MCP tool" into a
configuration task, not a code task.

## Lessons learned

1. **Schema validation at the boundary is non-negotiable.** LLMs will
   send malformed input. Validate before hitting the real API.
2. **Auth must be abstracted.** API keys, OAuth, JWT, custom headers —
   adapters should handle all of them so the agent never sees credentials.
3. **Async by default.** Sequential tool calls kill agent throughput.
4. **Error messages should be agent-readable.** "401 Unauthorized" is
   useless to an LLM. "API key invalid or expired — please rotate" is
   actionable.