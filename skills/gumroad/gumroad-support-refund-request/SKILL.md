---
name: gumroad-support-refund-request
description: >
  Handle buyer refund requests on Gumroad — standard, subscription, and duplicate-charge
  cases. Use when a ticket asks for money back, reports a double charge, or a seller is
  unresponsive to a refund. Decides eligibility, executes, and replies.
---

# Gumroad Support — Refund Request

Handle a buyer's refund request end-to-end: verify the purchase, decide eligibility
against policy, execute the refund, log it, and reply in Gumclaw's voice.

## Rule 0 (PRE) — eligibility gate

A refund is issued ONLY for: billing error · fraud · duplicate charge · seller-unresponsive.
Never refund our-pocket on a valid charge for a delivered product. There is no dollar cap,
but the reason must be real and grounded in the ticket + purchase record.

## Steps

1. Pull the purchase by email / card-last-4 / purchase ID.
   ```
   ⟨redacted: prod lookup command⟩
   ```
2. Confirm the charge state and whether a refund already exists.
3. Classify the request against Rule 0. If it fails the gate, explain why and offer the
   real path (contact seller, dispute, etc.) instead of refunding.
4. Execute the refund:
   ```
   ⟨redacted: admin refund command — requires operator auth⟩
   ```
   If the balance is insufficient, use the `--force` path ONLY for a genuine
   billing-error / fraud case (never to bypass Rule 0).
5. Log an admin comment on the account recording the reason and outcome.
6. Reply to the buyer. No sign-off, no apology theater, no em-dashes, no emoji.

## Pitfalls

- Helper AUTO-REOPENS on a customer reply — always re-pull the latest ticket state and act
  on the newest message before closing.
- A duplicate-charge refund is the SECOND charge only; never refund the legitimate one.
- Subscription refunds: decide whether to also cancel the subscription, or only refund the
  disputed period.

## Verify

- Refund shows as `refunded` in the live purchase record (read it back — do not trust the
  command's own success message).
- Admin comment is logged.
- Ticket closed (or escalated with a clear reason).

<!-- ⟨redacted⟩ — exact commands, endpoints, and auth flow removed for public showcase -->
