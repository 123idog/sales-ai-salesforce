# AI and Automation on Salesforce for Sales Teams

**What I learned connecting AI agents, messaging platforms and automation to a live Salesforce org used by a sales team every day.**

Salesforce lets an integration *write* almost anything. The hard part is that the org fights back: assignment rules, Flows, duplicate rules and restricted picklists silently change or reject what you send. Many failures return success. This repo collects the patterns that kept AI-driven writes correct, attribution trustworthy and sales reps in control.

---

## Contents

| # | Chapter | What's inside |
|---|---|---|
| 1 | [The Org Fights Back](docs/01-org-fights-back.md) | Assignment rules and Flows that overwrite your writes, converted leads, restricted picklists, REST gaps |
| 2 | [Lead Intake and Attribution](docs/02-lead-intake-and-attribution.md) | Phone normalisation everywhere, stale UTM defaults, entry point vs channel, real keywords only |
| 3 | [AI Writing to the CRM](docs/03-ai-writing-to-crm.md) | Matching records by phone, guarded status changes by a bot, bot vs human attribution, filter before the LLM |
| 4 | [Real-Time Triggers](docs/04-real-time-triggers.md) | Push, not poll; Outbound Message ACKs and lag; Flow activation; conversion-API flows |
| 5 | [Alerts and Owner Sync](docs/05-alerts-and-owner-sync.md) | Hot-lead pop-ups for reps, syncing ownership with a messaging platform without undoing manual changes |
| 6 | [Bulk Operations Without Regret](docs/06-bulk-operations.md) | Exact IDs, COUNT() first, side effects of bulk updates, silent query caps |
| 7 | [Analytics on CRM Data](docs/07-analytics-on-crm-data.md) | Check fields hold data, cross-system matching, correlation vs causation |

---

## Six rules

1. **A 2xx response isn't proof.** Read the record back after every write that matters.
2. **Assume the org will rewrite your fields.** Create, then PATCH what automation overwrote.
3. **Filter converted leads out of every write.** They can't be updated, and they'll fail half a backfill.
4. **Push status changes; don't poll for them.** Polling misses transitions.
5. **A bot may set a status; a human's status is final.** AI changes have guards, a written reason and an undo path.
6. **COUNT() before you write, test on five, then run.** Target exact IDs, never a heuristic filter.

---

## About

Written by **Ido Guy**, AI & Automation Engineer. I design end-to-end AI and CRM automation solutions for sales and operations teams.

[LinkedIn](https://www.linkedin.com/in/ido-guy) · Also: [WhatsApp AI Agent Design](https://github.com/123idog/whatsapp-ai-agent-design) · [LLM Evaluation in Production](https://github.com/123idog/llm-evaluation-in-production) · [n8n Production Playbook](https://github.com/123idog/n8n-production-playbook)

*All examples are generalised and numbers are rounded. No client data, schema, prompts or identifiers are included.*
