---
name: gumroad-risky-diff-review
description: >
  Run the adversarial pre-merge panel on any diff that touches money, payouts, refunds,
  fraud detection, access control or customer data. The verdict is pinned to the exact head
  sha; a new push needs a new verdict. Use before merging, not after.
---

# Gumroad Engineering — Risky Diff Review (panel pinned to the sha)

Most diffs merge on CI plus one review. A *risky* diff — one that can move money, change who
gets paid, loosen a fraud check or widen access — gets an adversarial panel first. The
panel's output is a verdict for one commit. "Clean at ⟨sha⟩" is the only form a verdict takes.

## Rule 0 (PRE) — what makes a diff risky

Any of: payout logic, refund or credit issuance, fee or tax calculation, fraud/risk scoring,
suspension or block logic, authentication or authorization, data export or erasure,
migrations on money tables, feature flags that gate any of the above. When unsure, treat it
as risky; the panel is cheap compared to a wrong payout.

## Steps

1. Pin the head. Record the PR's current head sha. Every artifact below names it.
   ```
   ⟨redacted: fetch PR head sha⟩
   ```
2. Build the review packet: the full diff, the PR description, the linked issue, the list of
   specs added or changed, and the migration plan if any. No chat transcripts, no customer
   data; the panel sees code and intent.
3. Run the panel. Three independent readers plus two tools, in parallel:
   - a strong frontier model prompted as an adversary ("how does this lose money or let a
     bad actor through?");
   - a second, different model family with the same prompt, so one vendor's blind spot is
     not the panel's blind spot;
   - an automated code-review service over the diff;
   - the house style linter (naming, structure, test shape);
   - the CI run for the same sha.
   ```
   ⟨redacted: panel runner — takes repo, PR number, head sha; writes a verdict file⟩
   ```
4. Triage findings. Each finding is kept, downgraded or dismissed with a one-line reason.
   A finding is not dismissed because it is inconvenient; it is dismissed because it is
   wrong, with the evidence (a spec, a line of code) that shows it is wrong.
5. Fix what is kept. Push the fix. **This moves the head sha — go back to step 1.** A verdict
   for the old sha is now history, not a pass.
6. Record the verdict: `clean at ⟨sha⟩` or `blocked at ⟨sha⟩: ⟨finding⟩`, in the PR's single
   agent comment (edit it; do not stack comments while no human has replied) and in the
   verdict file.
7. Merge only when the verdict sha equals the head sha and CI for that sha is green. A
   pre-merge gate script checks both and refuses otherwise.
   ```
   ⟨redacted: merge gate — compares verdict sha, head sha, CI status⟩
   ```
8. After merge, the daily security review loop re-reads the merged diff with a fresh
   prompt. That is the backstop, not a substitute for the panel.

## Verify

- Verdict file exists and names the same sha as the PR head at merge time.
- Every kept finding has a fixing commit; every dismissed finding has a stated reason.
- CI is green for the verdict sha, not just "latest".
- The PR carries exactly one agent comment, last edited to the final verdict.
- The merge gate ran and passed (its output is in the PR note).

## Pitfalls

- "I re-ran the panel mentally after the small fix" is not a verdict. Re-run it.
- Rebases change the sha even when the diff is identical. The gate is strict on purpose;
  re-run the panel rather than teaching the gate exceptions.
- Panel models agreeing with each other is not evidence. Two independent readers is the
  point; if both are from one vendor, the panel is one reader.
- Do not feed the panel the customer thread. It biases toward "ship it for the customer"
  and leaks data into a model prompt for no gain.
- A risky diff with no new spec is blocked by default. The panel does not accept "covered
  by existing tests" without the test name.
- Feature-flag rollouts for money paths go to everyone at once, with a replayable check, and ramp
  *down* on a trip. A staircase rollout only delays finding out.

<!-- ⟨redacted⟩ — panel runner, model pins, gate script and repo paths removed for public showcase -->
