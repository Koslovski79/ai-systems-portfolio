# MCP Security Platform

A reference implementation of the adapter pattern over MCP (Model Context
Protocol). Wrap any REST or GraphQL API as tools that an LLM agent can call
through a uniform interface.

**Public repository:** [platform-adapter-mcp](https://github.com/Koslovski79/platform-adapter-mcp)
**Status:** Active development

---

## What It Does

Exposes MCP servers as adapters over external security platforms. The same
agent loop that calls one tool can call any wrapped API — with auth, rate
limiting, and error handling normalised at the adapter layer.

This lets agents operate against platforms that were never designed for
programmatic LLM access. They get clean tool semantics, schema validation,
and async execution without each integration re-inventing the same boilerplate.

**Adapter targets include:** Caido proxy, Kali tooling, REST security APIs,
and GraphQL-based platforms.

---

## Architecture

```
  +-----------------+        +---------------------+
  |   LLM Agent     | stdio  |   MCP Server        |
  |  (Hermes,       | <----> |                     |
  |   Claude Code,  | JSON   |  +---------------+  |
  |   OpenCode)     | RPC    |  |  Tool Registry|  |
  +-----------------+        |  +-------+-------+  |
                             |          |           |
                             |  +-------v-------+  |
                             |  |   Adapters    |  |
                             |  | (REST/GraphQL |  |
                             |  |  /CLI/DB)     |  |
                             |  +---------------+  |
                             +---------------------+
```

---

## Stack

| Layer | Technology |
|---|---|
| Core framework | Python 3.11+ |
| Tool integration | MCP (Model Context Protocol) |
| Concurrency | asyncio |
| HTTP clients | httpx / aiohttp |
| Schema validation | Pydantic |

---

## Key Design Decisions

**MCP is becoming the standard agent-to-tool interface.**
Most real-world APIs weren't built for LLM access — they have auth quirks,
rate limits, unusual schemas, and paginated responses. This platform turns
"wrap a REST endpoint as an MCP tool" into a configuration task, not a
code task.

**Schema validation at the boundary is non-negotiable.**
LLMs will send malformed input. Validate before hitting the real API.

**Auth must be abstracted.**
API keys, OAuth, JWT, custom headers — adapters handle all of them so the
agent never sees raw credentials.

**Error messages must be agent-readable.**
`401 Unauthorized` is useless to an LLM. `API key invalid or expired —
please rotate` is actionable.

---

## Lessons Learned

1. **Schema validation at the boundary is non-negotiable.** Malformed LLM output hitting a real API causes hard-to-debug failures downstream.
2. **Auth abstraction protects the agent loop.** Credentials belong in the adapter, not in prompts or tool responses.
3. **Async by default.** Sequential tool calls kill agent throughput at scale.
4. **Error messages should be agent-readable.** Design error responses for the LLM consumer, not the human debugger.
