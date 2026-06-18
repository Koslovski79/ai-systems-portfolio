# AI Agent Platform

A modular Python framework for orchestrating prompt-driven AI agents with
persistent multi-layer memory. Runs against local open-weighted LLMs
(Ollama) by default, but works with any OpenAI-compatible backend.

**Public repository:** https://github.com/Koslovski79/prompt-orchestrator

## What it does

Breaks a complex request into a phase tree (`discover -> scan -> analyze
-> build -> integrate -> document`), routes each phase to a specialist
agent, and persists intermediate state to memory so workflows can pause,
resume, and survive session restarts.

The orchestrator, planner, and worker agents coordinate through a shared
memory layer. Skills are plug-in modules that wrap external CLI tools,
HTTP APIs, or local actions behind a uniform interface.

## Architecture

```
                +-------------------+
                |     User prompt   |
                +---------+---------+
                          |
                          v
                +-------------------+
                |   Planner agent   |  (decompose to phase tree)
                +---------+---------+
                          |
                          v
                +-------------------+
                | Orchestrator      |  (route phases, manage state)
                +---------+---------+
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
        +---------+ +---------+ +---------+
        | Discover| |  Scan   | | Analyze |   (specialist workers)
        +---------+ +---------+ +---------+
              |           |           |
              +-----------+-----------+
                          |
                          v
                +-------------------+
                |   Memory layer    |  (SQLite + FAISS + vector)
                +-------------------+
```

## Stack

- **Python 3.11** — core framework
- **SQLite** — structured state, knowledge base, session log
- **FAISS** — vector memory for semantic recall
- **Ollama** — default LLM backend (local, open-weighted)
- **MCP** — tool integration via the Model Context Protocol
- **OpenCode plugins** — extend agent capabilities

## Lessons learned

1. **Persistent memory is the single biggest unlock.** Without it, every
   session starts cold. With it, agents can resume weeks-old workflows,
   recall past findings, and improve over time.
2. **Phase decomposition beats monolithic prompts.** A 6-phase tree
   produces better results than one giant prompt asking the LLM to do
   everything. Each phase has a tight scope, explicit inputs and outputs,
   and a clear handoff.
3. **Skill registries beat hard-coded tools.** A registry pattern lets
   you add new tools without touching the orchestrator. Domain-agnostic
   by design.
4. **Resume and drift detection matter in production.** Workflows run
   for hours or days; they must survive crashes, network outages, and
   schema migrations.