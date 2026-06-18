# AI Systems Portfolio

A portfolio of working systems I've built and operated.

I don't open-source the code here. The point is to **prove the systems
exist** — the architecture, what they do, what I learned building them.

## Systems

| System | What it is | Stack | Status |
|---|---|---|---|
| [AI Agent Platform](./ai-agent-platform/) | Modular framework for orchestrating prompt-driven AI agents with persistent multi-layer memory | Python · SQLite · FAISS · Ollama | Public: [prompt-orchestrator](https://github.com/Koslovski79/prompt-orchestrator) |
| [MCP Security Platform](./mcp-security-platform/) | Adapter pattern over MCP — wrap any REST/GraphQL API as tools an LLM agent can call | Python · MCP · asyncio | Public: [platform-adapter-mcp](https://github.com/Koslovski79/platform-adapter-mcp) |
| [N8N Workflows](./n8n-workflows/) | Self-hosted N8N security automation with local LLM inference | N8N · Ollama · Docker | Public: [n8n-security-automation](https://github.com/Koslovski79/n8n-security-automation) |
| [Payroll System](./payroll-system/) | Full-stack payroll for South African SMEs, SARS-compliant | Django 4.2 · PostgreSQL | Private (production) |
| [POS System](./pos-system/) | Restaurant POS with 46 models, kitchen + table management | Django 4.2 · PostgreSQL | Private (production) |

## What this proves

Each system was a real problem I owned end-to-end:
- Designed the architecture
- Built and deployed it
- Operated it under real load
- Fixed the things that broke

The pattern: take a messy real-world problem → turn it into a working system.

## Contact

GitHub: [@Koslovski79](https://github.com/Koslovski79)