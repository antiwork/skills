# Changelog

All counts are measured on the live machine on the snapshot date, by script. Private
material is redacted per the rules in `.gitignore`.

## 2026-11-08 — second snapshot

- `README.md`: badges and layer table refreshed (487 skills, 191 cron loops, 1,288 scripts);
  domain table rebuilt from today's 25 categories; new "Model routing", "What changed since
  July 2026" and "Example skills" sections; guardrails list extended with the rules added
  since July (one comment per PR, ownership contract, access boundaries, public-artifact
  scrub, measure-before-claiming, comment gate, merged ≠ live, panel pinned to sha).
- `crons/MANIFEST.txt`: rebuilt from the live scheduler; lists the 128 standing loops by
  domain (support, risk & fraud, trust & safety, finance & accounting, tax, engineering &
  deploys, growth & marketing, infrastructure & self-preservation, comms). Case-scoped
  follow-ups are omitted.
- `skills/gumroad/gumroad-ledger-close/SKILL.md`: new redacted example — daily and monthly
  double-entry ledger close.
- `skills/gumroad/gumroad-deploy-verify/SKILL.md`: new redacted example — ship, wait for the
  real deploy, read back the served revision, then close the customer loop.
- `skills/gumroad/gumroad-risky-diff-review/SKILL.md`: new redacted example — pre-merge panel
  on money/fraud diffs with the verdict pinned to the head sha.
- `CHANGELOG.md`: added.

## 2026-07-01 — initial showcase

- First public, redacted snapshot: `README.md` (273 skills, 78 cron loops, 85 scripts),
  `crons/MANIFEST.txt`, one example skill (`gumroad-support-refund-request`), and the
  redaction `.gitignore`.
