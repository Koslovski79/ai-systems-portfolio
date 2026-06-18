# N8N Workflows

Self-hosted N8N instance running security and automation workflows with
local LLM inference. Webhook-triggered, scheduled, and event-driven.

**Public repository:** https://github.com/Koslovski79/n8n-security-automation

## What it does

N8N orchestrates a pipeline of workflows that:
- Receive alerts from external sources (email, webhooks, scheduled triggers)
- Route them through classification and enrichment steps
- Use a local LLM (Ollama) for analysis and summarization
- Push results to Slack, Discord, email, or downstream services

Everything runs on a self-hosted Docker stack. No data leaves the host.

## Architecture

```
  +-----------+        +-------------+        +-------------+
  |  Triggers |  ----> |   N8N core  | ---->  |   Ollama    |
  | (webhook, |        |  (workflows,|        |   (local    |
  |  email,   |        |   routing,  |        |    LLM)     |
  |  cron)    |        |   retries)  |        +-------------+
  +-----------+        +------+------+               |
                              |                      v
                              |              +-------------+
                              +------------> |  Classifier |
                              |              +-------------+
                              |                      |
                              v                      v
                       +-------------+       +-------------+
                       |  Routers    |       |  Output     |
                       |  (per-      |       |  channels   |
                       |   channel)  |       | (Slack,     |
                       +-------------+       |  Discord,   |
                                            |  email)     |
                                            +-------------+
```

## Stack

- **N8N** — workflow orchestration (self-hosted)
- **Ollama** — local LLM inference (open-weighted models)
- **Docker Compose** — service orchestration
- **Redis** — queue + state
- **PostgreSQL** — N8N's persistent store
- **Tailscale** — secure remote access without exposing ports

## Workflow categories

- **Recon pipelines** — automated discovery and surface mapping
- **Alert enrichment** — context for incoming security alerts
- **Report generation** — periodic summaries sent to channels
- **CI/CD hooks** — N8N receives events from build systems

## Lessons learned

1. **Self-hosting N8N gives full control over data.** SaaS workflow
   tools log everything; for sensitive automation, that's a non-starter.
2. **Local LLM routing keeps cost predictable.** Pay once for the GPU,
   run unlimited inferences.
3. **Webhook + scheduled + event triggers cover 95% of automation needs.**
   Most "complex" workflows are just a few of these chained together.
4. **N8N's UI lowers the barrier for non-developers.** Operators can
   modify workflows without touching code.