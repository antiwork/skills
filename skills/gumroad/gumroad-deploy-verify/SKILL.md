---
name: gumroad-deploy-verify
description: >
  Ship a fix, wait for the real production deploy, read back the served revision, and only
  then tell the customer or close the ticket. Use whenever a support ticket, bug report or
  seller promise ends in "we fixed it". Merged is not live.
---

# Gumroad Engineering — Deploy Verify (ship → wait → read back → close)

The failure this skill prevents: a PR merges, the agent tells the customer "this is fixed",
and production is still serving the old build for another hour — or the deploy failed.
The customer loop closes only after production itself says the fix is there.

## Rule 0 (PRE) — three different facts

- **Merged**: the commit is on `main`. Says nothing about production.
- **Deployed**: the deploy pipeline reports success for that commit.
- **Live**: a request to production returns the new revision marker *and* the new behavior.

Only "live" earns a customer reply or a ticket close. Do not shorten the chain.

## Steps

1. Land the fix. Non-feature, non-money fixes go straight to `main` once local specs pass.
   Features and anything touching money go through a PR and, if risky, the review panel
   (see `gumroad-risky-diff-review`). Record the merge sha; everything below is keyed to it.
2. Take the deploy lock. Two lanes must not ship at once; the lock names the lane and the
   sha, and a stale lock is visible to the janitor loop.
   ```
   ⟨redacted: deploy lock acquire — lane id + sha⟩
   ```
3. Wait on the dynamic gate, not a fixed sleep. The gate polls the deploy system for the
   sha and returns when the deploy for *that sha* finishes (success or failure). A fixed
   `sleep 600` is wrong in both directions: too short on a slow build, too long on a fast one.
   ```
   ⟨redacted: deploy gate — waits for the pipeline run of this sha⟩
   ```
4. Read back the served revision. Production exposes its revision (an `x-revision`-style
   header or a version marker). Compare it to the merge sha. If they differ, the deploy did
   not land — stop, do not notify anyone, and investigate.
   ```
   ⟨redacted: served-revision probe against the production host⟩
   ```
5. Read back the behavior. The revision proves the code is there; a behavior probe proves
   the fix works for the case that was reported. Use the real page or endpoint the customer
   hit, not an HTML fixture.
6. Release the deploy lock.
7. Now close the customer loop: reply on the ticket, close it, update the tracker. The
   reply states what changed in plain words; it never contains the sha or internal names.
8. Post-deploy impact monitoring runs for a window after any deploy. If error rates move,
   the impact loop re-opens the ticket and pages the lane before the customer notices.

## Verify

- The served revision equals the merge sha (copied from the probe, not recalled).
- The behavior probe passed on the production host.
- The deploy lock is released; the janitor sees no stale lock for this lane.
- The ticket close carries the probe result in its internal note.

## Pitfalls

- A green CI run on the PR is not a deploy. CI tests the branch; the gate watches production.
- Revision read-back from a cached or CDN-fronted path can show the new revision while an
  uncached path still serves old code. Probe the path the customer actually used.
- Rollbacks move the served revision *backwards*. A close that happened before a rollback is
  now wrong; the impact loop must re-open it.
- Do not "close as fixed" on a promise. If the deploy will land later, the ticket stays open
  with a dated follow-up loop attached (that loop retires when the close happens).
- If the deploy is stuck waiting on human approval, that is a `[gated]` state. Say so on the
  ticket; do not sit silently.
- Never tell a customer a time ("in 20 minutes") you do not control. Tell them when it is live.

<!-- ⟨redacted⟩ — probe commands, deploy system endpoints, lock location and hostnames removed for public showcase -->
