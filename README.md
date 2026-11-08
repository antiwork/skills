<div align="center">

# 🦾 Gumclaw

**The autonomous brain that runs 99.99% of Gumroad.**

Support · Risk · Fraud · Trust & Safety · Finance · Compliance · Engineering.
_Everything but paying taxes._

![skills](https://img.shields.io/badge/skills-487-3fb950)
![cron loops](https://img.shields.io/badge/cron_loops-191-2f81f7)
![scripts](https://img.shields.io/badge/scripts-1288-bc8cff)
![runtime](https://img.shields.io/badge/runtime-Hermes_Agent-db6d28)
![uptime](https://img.shields.io/badge/uptime-24%2F7-3fb950)

**→ [See the architecture](https://antiwork.com/skills) ←**

</div>

> ⚠️ **This repository is a redacted architectural showcase.** Every credential, prod endpoint, customer record, seller identity, ticket ID, and chat destination has been removed or replaced with `⟨redacted⟩`. It exists to explain *how* the brain is built — not to be run. Nothing here is executable or reconstructable.

> Snapshot date: **2026-11-08**. Every count on this page was measured on the live machine that day by a script, not recalled. See [`CHANGELOG.md`](CHANGELOG.md) for what moved since the first snapshot in July 2026.

---

## What this is

Gumclaw is a single autonomous agent, running on **Hermes Agent**, that operates a real business. Not a chatbot bolted onto a help desk — the actual operator. It reads the support queue, investigates fraud, issues refunds, closes tickets, adjudicates piracy reports, writes and ships code, reads the deploy back before it tells anyone it shipped, closes the books on a double-entry ledger, prepares tax filings, and pings a human only when a decision genuinely needs one.

The "brain" is three layers:

| Layer | Count | What it is |
| --- | --- | --- |
| **Skills** | 487 | Procedural memory — how to do the work |
| **Cron loops** | 191 | Autonomous cadence — the heartbeat (128 standing, the rest case-scoped) |
| **Memory** | 2 files | Durable state — what it knows and who it serves |

Behind them sit **1,288 helper scripts** (Python, shell, Ruby) that do the mechanical parts — pulling a queue, checking a deploy, reconciling a ledger — so the model reasons about results instead of re-deriving them.

## Architecture

```
                         ┌───────────────────────────────────────────┐
                         │   INPUTS  (the world talks to Gumclaw)    │
                         └───────────────────────────────────────────┘
   Helper support queue ─┐   GitHub webhooks ─┐   risk / finance mail ┐   team chat ─┐
   Gmail push + poll ────┤   Sentry issues ───┤   CSAT scans ─────────┤   cron ticks ─┤
   Stripe / PayPal / bank┤   CI status ───────┤   piracy & DMCA ──────┤   deploy hooks┤
                         ▼                     ▼                       ▼              ▼
              ╔═══════════════════════════════════════════════════════════════════╗
              ║                      GUMCLAW  ·  Hermes Agent                     ║
              ║   SOUL.md / PERSONALITY   MEMORY.md / USER.md   SKILLS ×487        ║
              ║   who I am                what I know           how to do the work ║
              ║                 SCRIPTS ×1288  ·  CRON LOOPS ×191                  ║
              ║                     REASON → PLAN → ACT → VERIFY                    ║
              ╚═══════════════════════════════════════════════════════════════════╝
                         │                     │                │
                         ▼                     ▼                ▼
              AUTONOMOUS               GATED                HUMAN
              reply · refund · close   ship money/fraud     pay taxes
              suspend fraud · KYC      code (panel first)   irreversible spend
              ledger close · deploys   payouts · publishing one-way doors
                         │                     │
                         └──────────┬──────────┘
                                    ▼
                      CRITIC → BACKSTOP → SWEEP → READ-BACK
                      (every action re-verified by an independent pass on a timer;
                       "shipped" means the served revision was read back, not merged)
```

## The three layers

### 1 · Skills — procedural memory (487)

Each skill is a `SKILL.md`: a trigger, a numbered procedure, exact commands, pitfalls, and verification steps. When a task matches, the skill loads and Gumclaw follows the house method — not an improvised guess. This is how one agent holds the expertise of an entire ops + support + finance + eng team. Skills that stop earning their keep are archived, not deleted: 235 retired ones sit in an archive and are not counted above.

| Domain | Skills | Covers |
| --- | --- | --- |
| **gumroad** | 319 | Support triage, risk/fraud, refunds, KYC, payouts, trust & safety, GDPR erasure, DMCA, ledger close, tax filings, mobile release, engineering |
| software-development | 26 | Planning, systematic debugging, code review, flag-gated shipping, post-merge regression triage, subagent-driven dev |
| devops | 22 | CI, deploy pipeline + lock, webhooks, infra hygiene, kanban orchestration, SDLC review |
| github | 19 | Issue-to-PR, review evidence, merge gates, ownership contract, tracker stewardship |
| productivity | 18 | Google Workspace, Notion, Linear, Ramp, calendars, e-signature, OCR |
| gumroad-ops | 14 | Operator-CLI guard rails, backfill attribution, destination verification, vendor payments |
| creative | 13 | Diagrams, illustration, pixel art, short-film pipelines, one-shot HTML artifacts |
| hermes | 8 | Cron fleet design, worker lanes, runtime self-repair, inbound agent endpoints |
| _+ 17 more domains_ | 48 | autonomous-ai-agents, research, security, payments, social-media, apple, data-science, web, mcp, comms, media, office, mlops, email… |

(25 top-level categories in total; the eight above hold 439 skills, the other 17 hold 48.)

### 2 · Cron loops — autonomous cadence (191)

The brain doesn't wait to be asked. 191 enabled recurring jobs give it a heartbeat: **128 standing loops** (support sweeps, fraud scans, ledger close, deploy gates, watchdogs) plus about 63 **case-scoped follow-ups** — a loop attached to one open ticket or PR that retires when it closes. Cadences range from every 2 minutes (watchdogs) to every 2 days; 130 are cron expressions, 61 are fixed intervals. Most run **silent**, speaking up only when something is worth a human's attention, and a daily message budget caps how often the scheduler may interrupt a person at all. The standing loops are listed in [`crons/MANIFEST.txt`](crons/MANIFEST.txt).

### 3 · Memory — durable state

Two files persist across every session and are injected into every turn: `MEMORY.md` (environment facts, lessons, tool quirks) and `USER.md` (who the operator is, their preferences, their corrections). Both have a hard character budget, so a new lesson has to displace a stale one — the brain is forced to keep only what still applies. Combined with session search and automatic context compaction, it never re-learns the same lesson twice.

## Model routing

One agent, several models, chosen by the weight of the work:

| Lane | Model family |
| --- | --- |
| Volume work (support email, product and support investigation) | DeepSeek V4.1 Flash |
| Money, devops and design lanes | Claude Sonnet 5.5 |
| Review panels on risky diffs | Claude Opus 5.5 + an independent second model + Greptile + a house style linter |
| One-off, explicitly named builds | A frontier model chosen per task |

A written policy file says which lane gets which family. A daily drift-check loop compares every enabled job against it; an unpinned or off-policy job is re-pinned automatically and reported.

## What changed since July 2026

1. **Books moved off QuickBooks.** QuickBooks was retired in September. The ledger is now a self-hosted double-entry (Beancount) file with a daily close, a month close, bank and processor reconciliation, a tax-remittance reconciliation, and GAAP reviews on a monthly / quarterly / annual cadence. See [`gumroad-ledger-close`](skills/gumroad/gumroad-ledger-close/SKILL.md).
2. **Admin web UI deleted; operator CLI instead.** The internal admin pages went away in August. Every write the agent makes to the product is now one auditable CLI command with a per-command guard flag, instead of a click on a page nobody can replay.
3. **Review panels pinned to the commit.** A diff that touches money, payouts or fraud gets an adversarial panel before merge, and the verdict is recorded against the exact head sha. A new push invalidates the verdict. See [`gumroad-risky-diff-review`](skills/gumroad/gumroad-risky-diff-review/SKILL.md).
4. **Deploy read-back before "it shipped".** "Merged" is no longer accepted as "live". A deploy lock stops two lanes shipping at once, a dynamic gate waits for the real deploy instead of a fixed sleep, the served revision is read back, and only then is a customer told their fix is out. See [`gumroad-deploy-verify`](skills/gumroad/gumroad-deploy-verify/SKILL.md).
5. **Trust & safety grew real queues.** Piracy and DMCA adjudication runs hourly. Standing detectors cover card-testing rings, chargeback rings, invitation rings, refund abuse, dead-card checkouts and zero-deliverable sales bursts. A CSAM match protocol stops and escalates instead of acting.
6. **Support rails hardened.** A Gmail push rail with a poll backstop, per-requester sweeps, a session budget per ticket, a closed-ticket safety net that catches tickets closed in error, CSAT scanning, and a deploy-gated close: a ticket is not closed as "fixed" until production serves the fix.
7. **Cost governance.** A per-turn token ledger, a daily AI-spend digest, the model-policy drift check above, a hard daily cap on scheduler messages to humans, and a delivery ledger so a dropped report is visible instead of silently missing.
8. **Self-preservation watchdogs.** An off-host deadman heartbeat, a disk-headroom guard that protects every other job, a janitor for orphaned worker sessions, session garbage collection, state-DB pruning, memory reclaim and a nightly worktree sweep. The brain keeps its own machine healthy.
9. **Mobile shipping.** A weekly mobile app ship loop, a daily release loop, and a live notify that fires only once the release is actually live in the store.

## Guardrails

- **Confirm-before-every-payment.** No autonomous spend; vendor bank-detail changes trigger an out-of-band check.
- **Irreversible + ambiguous ⇒ ask.** One-way doors (money movement, publishing, account changes) are gated; a reversible action may proceed on its own.
- **Measure before claiming.** A number in a public artifact is computed by a script in the same session, not recalled. IDs and amounts are copied from a live source, never from memory.
- **Merged ≠ live.** Nothing is reported as shipped until the served revision is read back from production.
- **Review panel pinned to the sha.** A risky diff is clean *at a commit*, never in general; a new push needs a new verdict.
- **Never disable a feature to dodge a bug.** Fix the slow or broken thing; do not turn it off.
- **One comment per pull request** while no human has replied — edit the existing comment, never stack new ones. Bot and CI activity does not reset the count.
- **Comment gate.** Customer- and public-facing text goes through a length/structure gate: one to three sentences per point, cause → action, no checkboxes, no filler.
- **Ownership contract.** A draft PR belongs to the agent and carries a `working` label; `working` with no live session is visibly stuck. Human-decision items are labelled and assigned to the named human — the agent does not silently sit on them.
- **Access boundaries.** The agent does not widen its own access and does not self-grant permissions or credentials.
- **Public-artifact scrub.** Before anything is published: no customer or seller identity, no ticket slugs, no internal hostnames, no chat destinations, no credentials, no spend figures.
- **Critic + backstop passes.** Every loop is re-verified by an independent pass on a timer. "Done" means a critic confirmed it in production.
- **Humanization clause.** Every customer-facing line reads like a person wrote it.
- **Send-as attribution.** Human-in-the-loop replies go out under that human's name; autonomous ones under Gumclaw's.

## The one thing it doesn't do

> Gumclaw runs **99.99%** of Gumroad. The remaining 0.01% is **paying the taxes** — the one irreversible, judgment-and-liability call that stays with a human on purpose. It *prepares* everything (state sales tax, federal filings, treaty certificates, notice monitoring, remittance reconciliation); a person presses the final button.

## Example skills (redacted)

| Skill | What it shows |
| --- | --- |
| [`gumroad-support-refund-request`](skills/gumroad/gumroad-support-refund-request/SKILL.md) | The original example: eligibility gate, execute, read back, reply |
| [`gumroad-ledger-close`](skills/gumroad/gumroad-ledger-close/SKILL.md) | Daily and monthly double-entry ledger close |
| [`gumroad-deploy-verify`](skills/gumroad/gumroad-deploy-verify/SKILL.md) | Ship a fix, wait for the real deploy, read back the served revision, then close the customer loop |
| [`gumroad-risky-diff-review`](skills/gumroad/gumroad-risky-diff-review/SKILL.md) | The pre-merge panel on money/fraud diffs, verdict pinned to the head sha |

---

<div align="center">
Built on <b>Hermes Agent</b> by Nous Research · Operated by Antiwork<br>
<code>⟨redacted⟩</code> for public showcase — no secrets, no customer data, no reconstructable endpoints.
</div>
