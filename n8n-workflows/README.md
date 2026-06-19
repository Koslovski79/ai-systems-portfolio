# N8N Automation Workflows

Self-hosted N8N instance running security and business automation workflows
with local LLM inference. Webhook-triggered, scheduled, and event-driven.

**Public repository:** [n8n-security-automation](https://github.com/Koslovski79/n8n-security-automation)
**Status:** Active — running in production on self-hosted infrastructure

---

## What It Does

N8N orchestrates automation pipelines that:

- Receive inputs from external sources (email, webhooks, scheduled triggers)
- Route them through classification and enrichment steps
- Use a local LLM (Ollama) for analysis and summarisation
- Push results to output channels (Slack, Discord, email, downstream services)

Everything runs on self-hosted infrastructure via Tailscale. No data leaves
the host.

---

## Architecture

```
  +-----------+        +-------------+        +-------------+
  |  Triggers |  ----> |   N8N core  | ---->  |   Ollama    |
  | (webhook, |        | (workflows, |        |  (local LLM)|
  |  email,   |        |  routing,   |        +------+------+
  |  cron)    |        |  retries)   |               |
  +-----------+        +------+------+               v
                              |              +-------------+
                              |              |  Classifier |
                              |              +------+------+
                              |                     |
                              v                     v
                       +-------------+       +-------------+
                       |   Routers   |       |   Output    |
                       | (per-channel|       |  channels   |
                       |  logic)     |       | (Slack,     |
                       +-------------+       |  Discord,   |
                                            |   email)    |
                                            +-------------+
```

---

## Stack

| Layer | Technology |
|---|---|
| Workflow orchestration | N8N (self-hosted) |
| LLM inference | Ollama (local, open-weighted) |
| Secure remote access | Tailscale |
| Queue + state | Redis |

---

## Workflow Categories

**Recon pipelines**
Automated discovery and surface mapping. Triggered on schedule or manually
via webhook.

**Alert enrichment**
Incoming security alerts are classified, enriched with context, and
summarised by the local LLM before routing to the relevant channel.

**Report generation**
Periodic summaries (daily/weekly) generated and delivered automatically.

**CI/CD hooks**
N8N receives build and deployment events and routes notifications or
triggers downstream actions.

---

## Key Design Decisions

**Self-hosting for data control.**
SaaS workflow tools log everything. For security automation, that's a
non-starter. Full control over data means full control over what leaves
the host — which is nothing.

**Local LLM for cost predictability.**
Pay once for the infrastructure, run unlimited inferences. No per-token
costs on high-volume automation.

**Webhook + scheduled + event triggers.**
These three patterns cover the vast majority of automation requirements.
Most complex-looking workflows are simple chains of these primitives.

---

## Lessons Learned

1. **Self-hosting gives full data control.** For sensitive automation, SaaS tools that log all workflow data are a liability.
2. **Local LLM inference keeps costs flat.** High-volume automation against cloud APIs gets expensive fast.
3. **Three trigger types cover almost everything.** Webhook, schedule, and event — master these and you can automate most operational problems.
4. **N8N's visual interface lowers the barrier for operators.** Non-developers can read, modify, and build workflows without touching code.
