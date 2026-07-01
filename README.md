<div align="center">

# 🦾 Gumclaw

**The autonomous brain that runs 99.99% of Gumroad.**

Support · Risk · Fraud · Finance · Compliance · Engineering.
_Everything but paying taxes._

![skills](https://img.shields.io/badge/skills-273-3fb950)
![cron loops](https://img.shields.io/badge/cron_loops-78-2f81f7)
![scripts](https://img.shields.io/badge/scripts-85-bc8cff)
![runtime](https://img.shields.io/badge/runtime-Hermes_Agent-db6d28)
![uptime](https://img.shields.io/badge/uptime-24%2F7-3fb950)

**→ [See the architecture](https://antiwork.com/skills) ←**

</div>

> ⚠️ **This repository is a redacted architectural showcase.** Every credential, prod endpoint, customer record, seller identity, ticket ID, and chat destination has been removed or replaced with `⟨redacted⟩`. It exists to explain *how* the brain is built — not to be run. Nothing here is executable or reconstructable.

---

## What this is

Gumclaw is a single autonomous agent, running on **Hermes Agent**, that operates a real business. Not a chatbot bolted onto a help desk — the actual operator. It reads the support queue, investigates fraud, issues refunds, closes tickets, writes and ships code, reconciles the books, files tax reports, and pings a human only when a decision genuinely needs one.

The "brain" is three layers:

| Layer | Count | What it is |
| --- | --- | --- |
| **Skills** | 273 | Procedural memory — how to do the work |
| **Cron loops** | 78 | Autonomous cadence — the heartbeat |
| **Memory** | 2 files | Durable state — what it knows and who it serves |

## Architecture

```
                         ┌───────────────────────────────────────────┐
                         │   INPUTS  (the world talks to Gumclaw)    │
                         └───────────────────────────────────────────┘
   Helper support queue ─┐   GitHub webhooks ─┐   risk@ inbox ─┐   Telegram DMs ─┐
   Stripe / QBO / Ramp ──┤   Sentry issues ───┤   CSAT scans ──┤   cron ticks ───┤
                         ▼                     ▼                ▼                ▼
              ╔═══════════════════════════════════════════════════════════════════╗
              ║                      GUMCLAW  ·  Hermes Agent                     ║
              ║   SOUL.md / PERSONALITY   MEMORY.md / USER.md   SKILLS ×273        ║
              ║   who I am                what I know           how to do the work ║
              ║                     REASON → PLAN → ACT → VERIFY                    ║
              ╚═══════════════════════════════════════════════════════════════════╝
                         │                     │                │
                         ▼                     ▼                ▼
              AUTONOMOUS               GATED                HUMAN
              reply · refund · close   ship code · payouts  pay taxes
              suspend fraud · KYC      public posts         irreversible spend
                         │                     │
                         └──────────┬──────────┘
                                    ▼
                      CRITIC → BACKSTOP → SWEEP
                      (every action re-verified by an independent pass on a timer)
```

## The three layers

### 1 · Skills — procedural memory (273)

Each skill is a `SKILL.md`: a trigger, a numbered procedure, exact commands, pitfalls, and verification steps. When a task matches, the skill loads and Gumclaw follows the house method — not an improvised guess. This is how one agent holds the expertise of an entire ops + support + finance + eng team.

| Domain | Skills | Covers |
| --- | --- | --- |
| **gumroad** | 146 | Support triage, risk/fraud, refunds, KYC, payouts, compliance, GDPR erasure, DMCA, tax reports, monthly close, engineering |
| software-development | 22 | TDD, systematic debugging, code review, planning, subagent-driven dev |
| creative | 20 | Diagrams, infographics, design artifacts, video, illustration |
| productivity | 14 | Google Workspace, Notion, Linear, Airtable, Ramp, PDFs |
| mlops | 13 | Fine-tuning, serving, evaluation, structured output |
| devops | 11 | CI, webhooks, key rotation, incident response, cost audits |
| _+ 15 more domains_ | 47 | media, github, apple, research, smart-home, email, gaming, social… |

### 2 · Cron loops — autonomous cadence (78)

The brain doesn't wait to be asked. 78 scheduled jobs give it a heartbeat — some every 2 minutes (watchdogs), some daily (risk, finance), some weekly (performance review, tax verification). Most run **silent**, speaking up only when something is worth a human's attention.

### 3 · Memory — durable state

Two files persist across every session and are injected into every turn: `MEMORY.md` (environment facts, lessons, tool quirks) and `USER.md` (who the operator is, their preferences, their corrections). Combined with session search and automatic context compaction, the brain never re-learns the same lesson twice.

## Guardrails

- **Confirm-before-every-payment.** No autonomous spend; vendor bank-detail changes trigger an out-of-band BEC check.
- **Irreversible-only gating.** Reversible actions run free; one-way doors are gated.
- **Critic + backstop passes.** Every loop is re-verified by an independent pass on a timer. "Done" means a critic confirmed it in production.
- **Anti-fabrication rule.** No fact asserted without a live source pulled this session. IDs and amounts are copied, never recalled.
- **Humanization clause.** Every customer-facing line reads like a person wrote it.
- **Send-as attribution.** Human-in-the-loop replies go out under that human's name; autonomous ones under Gumclaw's.

## The one thing it doesn't do

> Gumclaw runs **99.99%** of Gumroad. The remaining 0.01% is **paying the taxes** — the one irreversible, judgment-and-liability call that stays with a human on purpose. It *prepares* everything; a person presses the final button.

---

<div align="center">
Built on <b>Hermes Agent</b> by Nous Research · Operated by Antiwork<br>
<code>⟨redacted⟩</code> for public showcase — no secrets, no customer data, no reconstructable endpoints.
</div>
