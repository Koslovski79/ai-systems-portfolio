# AI Agent Platform

A modular Python framework for orchestrating prompt-driven AI agents with
persistent multi-layer memory. Runs against local open-weighted LLMs (Ollama)
by default, but works with any OpenAI-compatible backend.

**Public repository:** [prompt-orchestrator](https://github.com/Koslovski79/prompt-orchestrator)
**Status:** Active development

---

## What It Does

Breaks a complex request into a phase tree, routes each phase to a specialist
agent, and persists intermediate state to memory so workflows can pause, resume,
and survive session restarts.

The orchestrator, planner, and worker agents coordinate through a shared memory
layer. Skills are plug-in modules that wrap external CLI tools, HTTP APIs, or
local actions behind a uniform interface.

Default phase tree:

```
discover → scan → analyze → build → integrate → document
```

Each phase has a tight scope, explicit inputs and outputs, and a defined handoff
to the next phase. The orchestrator tracks state across all phases so a workflow
interrupted at `analyze` resumes at `analyze` — not from the beginning.

---

## Architecture

```
                +-------------------+
                |     User prompt   |
                +---------+---------+
                          |
                          v
                +-------------------+
                |   Planner agent   |  decompose to phase tree
                +---------+---------+
                          |
                          v
                +-------------------+
                |   Orchestrator    |  route phases, manage state
                +---------+---------+
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
        +---------+ +---------+ +---------+
        | Discover| |  Scan   | | Analyze |   specialist workers
        +---------+ +---------+ +---------+
              |           |           |
              +-----------+-----------+
                          |
                          v
                +-------------------+
                |   Memory layer    |  SQLite + FAISS + vector
                +-------------------+
```

---

## Stack

| Layer | Technology |
|---|---|
| Core framework | Python 3.11 |
| Structured state | SQLite |
| Vector memory | FAISS |
| LLM backend | Ollama (local, open-weighted) |
| Tool integration | MCP (Model Context Protocol) |

---

## Key Design Decisions

**Persistent memory is the core unlock.**
Without it, every session starts cold. With it, agents resume weeks-old
workflows, recall past findings, and improve over time. SQLite handles
structured state; FAISS handles semantic recall.

**Phase decomposition beats monolithic prompts.**
A 6-phase tree produces better results than one large prompt asking the LLM
to do everything. Each phase has a defined scope and clean handoffs.

**Skill registries beat hard-coded tools.**
A registry pattern lets you add new capabilities without touching the
orchestrator. Domain-agnostic by design — the same framework runs security
workflows, business automation, and development tasks.

**Resume and drift detection matter in production.**
Workflows run for hours or days. They must survive crashes, network outages,
and schema migrations.

---

## Lessons Learned

1. **Persistent memory is the single biggest unlock.** Cold-start agents lose context every session. Memory-backed agents compound knowledge over time.
2. **Phase decomposition beats monolithic prompts.** Tight scope per phase produces better LLM output and easier debugging.
3. **Skill registries beat hard-coded tools.** Adding a new capability should be a configuration task, not a refactor.
4. **Resume and drift detection are production requirements.** Long-running workflows will break. Build recovery in from the start.
