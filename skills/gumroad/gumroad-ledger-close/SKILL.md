---
name: gumroad-ledger-close
description: >
  Close the Gumroad books on the self-hosted double-entry ledger. Use for the daily close
  (feeds synced, balances tie, nothing unposted) and the month close (journal entries,
  reconciliation to bank and processors, review markers written). Replaces the retired
  QuickBooks workflow.
---

# Gumroad Finance — Ledger Close (daily + monthly)

The books live in one plain-text double-entry ledger (Beancount). Every processor payout,
bank movement, card charge and tax remittance becomes a balanced transaction in that file.
"Closed" means the ledger balances tie to every external source for the period and the
success marker for that period exists. No marker, no close.

## Rule 0 (PRE) — the ledger is the source of truth, feeds are inputs

Never hand-edit a balance to make it tie. If a balance does not tie, there is a missing or
duplicated transaction; find it. A close that "fixes" a gap with a plug entry is not a close.

## Daily close

1. Sync the feeds. One script pulls the day's processor balance transactions, bank
   transactions and card-spend lines into staging:
   ```
   ⟨redacted: ledger sync command — read-only API tokens, output to staging⟩
   ```
2. Import staging into the ledger. Each source has an importer that maps a raw line to a
   balanced posting. Unknown counterparties land in a "needs classification" account, never
   in a guessed expense account.
3. Run the balance assertions. The ledger carries a dated balance assertion per external
   account; the checker fails loudly on any mismatch:
   ```
   ⟨redacted: bean-check + balance assertion command⟩
   ```
4. Reconcile processors. For each payment processor, the day's settled payouts in the
   ledger must equal the processor's own payout report for the same day. Differences are
   either timing (settles tomorrow — note it) or a real miss (go back to step 2).
5. Reconcile the bank. Settled bank transactions vs ledger postings, by date and amount.
   The tax-authority rail is reconciled separately so a remittance can never be mistaken for
   an operating expense.
6. Write the success marker for the day. The hourly close watchdog looks for this marker;
   a missing marker is what pages the operator, not a chatty "all good" message.

## Month close

1. Confirm all daily markers for the month exist. A missing day blocks the month.
2. Post the month's journal entries: revenue recognition, processor fees, cost of goods,
   state sales-tax liabilities (one entry per jurisdiction), accruals, capital deductions.
   Each entry cites its source report in the narration so a reviewer can re-derive it.
3. Re-run balance assertions and the full ledger check across the month.
4. Produce the close pack: trial balance, P&L, balance sheet, reconciliation summary.
   ```
   ⟨redacted: report generation command — writes to the close folder⟩
   ```
5. The monthly GAAP review loop reads the close pack and raises findings as a numbered
   list (cause → action). Quarterly and annual reviews add their own checks on top.
6. Write the month marker. Post the close summary to the finance channel. The summary is
   a status with numbers copied from the pack, never typed from memory.

## Verify

- `bean-check` is clean and every dated balance assertion passes.
- Every processor and bank account ties for the period; timing items are listed, not hidden.
- The "needs classification" account is empty at month end.
- The period marker exists and the close watchdog is quiet.
- Every figure in the posted summary can be found verbatim in the close pack.

## Pitfalls

- A processor's "available" balance is not its "settled" balance; reconcile against settled.
- Tax remittances leave from a dedicated rail. If one shows up on the operating account,
  the rail top-up loop or the routing is wrong — do not just reclassify it.
- Refund reversals post on the refund date, not the original sale date.
- Multi-currency: convert at the processor's rate on the settlement line, not a daily
  average; the ledger keeps the original currency in the posting metadata.
- Do not run two closes at once. The close takes a lock; a second run waits.
- Paying the tax itself is a human action. The close prepares the liability and the
  remittance instruction; a person presses the button.

<!-- ⟨redacted⟩ — exact commands, account names, rails and file locations removed for public showcase -->
