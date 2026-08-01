## Context

See proposal.md ("Why"). The current implementation stores `amount_minor` with no currency and sums it directly, per the "Single currency across a group" requirement now being removed.

The interesting question in this change is not how to add a column. It is where to draw the boundary on conversion.

## Goals / Non-Goals

**Goals**

- Currency is a property of the expense, fixed at the moment it is recorded.
- Balances and settlement partition cleanly by currency, reusing the existing greedy algorithm unchanged.
- The migration is safe on a database where every existing expense predates currencies.

**Non-Goals**

- Exchange rates, conversion, or a "total in my home currency" view.
- Per-currency minor-unit precision. See Risks: this is a real gap being consciously deferred.
- Backwards compatibility of the API response shape. There are no external consumers yet.

## Decisions

### Currency on the expense, default on the group

`expenses.currency CHAR(3) NOT NULL`, `groups.default_currency CHAR(3) NOT NULL`. The group default is a convenience for the common case (a trip mostly spent in one place), applied at write time, not read time.

Applying it at write time matters: the currency is resolved once and stored, so a stored expense's meaning never depends on the group's current settings. That is what makes "Default currency is fixed after creation" enforceable.

*Alternative considered:* resolve the default at read time and leave `expenses.currency` nullable. Rejected: it makes every historical expense's meaning mutable, which is exactly the failure mode the fixed-default requirement exists to prevent.

### Partition, then reuse the existing algorithm

Settlement groups balances by currency and runs the existing greedy largest-creditor / largest-debtor matcher once per currency. No change to the matcher itself.

*Alternative considered:* a single cross-currency optimizer that minimizes total transfer count by converting. Rejected: it requires a rate, and a rate requires a date and a source. That is a product decision Splitr has no basis to make, and it is written into the specs as "No currency conversion" so that a future change has to argue for it explicitly.

### Refuse conversion loudly, not silently

Requesting balances with a `convertTo` parameter returns a validation error rather than being ignored. A silently ignored conversion parameter is worse than an unsupported one, because the client believes it worked.

### Migration backfills from the group default

Add both columns nullable, backfill `expenses.currency` from `groups.default_currency`, then set `NOT NULL`. `groups.default_currency` is backfilled to a deployment-time constant, since no group has expressed a preference.

*Alternative considered:* backfill to a hardcoded USD. Rejected as arbitrary; the deployment-time constant should be chosen by whoever runs the migration and recorded in the runbook.

## Risks / Trade-offs

- **Minor-unit precision is not uniform across currencies.** JPY has no minor unit; some currencies have three. The current code assumes two everywhere, and this change does not fix that. → Consciously deferred. It affects display and the remainder rule, not the balance arithmetic, since amounts are only ever compared within a currency. It needs its own change, and it should be raised before anyone records a JPY expense.
- **The API breaks.** → Acceptable now, since there are no external consumers. The window for this being free closes as soon as anything integrates.
- **Groups can accumulate many currencies, producing many small plans.** → Accepted. Reporting four two-line plans is more honest than one converted plan built on a rate nobody agreed to.
- **The greedy-vs-provably-minimal divergence from the previous change is inherited unchanged**, now per currency. → Still open. Still not made worse by this change.

## Migration Plan

1. Add `groups.default_currency` and `expenses.currency`, both nullable.
2. Backfill `groups.default_currency` to the constant chosen in the runbook.
3. Backfill `expenses.currency` from the owning group's default.
4. Set both `NOT NULL`.
5. Deploy the API change.

Rollback: drop the two columns. The old code ignores them, so steps 1 through 4 are safe to leave in place if step 5 is reverted.

## Open Questions

- Should the balances response list currencies in a stable order (for example, alphabetically by code) so clients can diff two responses? Deferrable: it does not change any requirement, and no scenario depends on ordering.
