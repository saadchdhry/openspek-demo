## Why

Splitr currently has no way to correct a recorded expense. The `expense-recording` capability records an expense and never revisits it, which was a deliberate simplification in `add-expense-splitting` and is now the most common complaint: somebody fat-fingers 4500 instead of 450 and the group's balances are wrong forever.

The awkward part is not editing. It is that a settlement plan someone already acted on may have been computed from the wrong number.

## What Changes

- Allow correcting an expense's description, amount, currency, payer, and participants.
- Preserve the original as an immutable revision rather than overwriting it.
- Allow voiding an expense, which excludes it from balances without deleting it.
- Report on the balances response whether any expense has been edited since the group last had a zero balance.

## Capabilities

### New Capabilities

- `expense-revisions`: the revision history of an expense, and what it means for an expense to be superseded or voided.

### Modified Capabilities

- `expense-recording`: recording an expense now creates revision 1 rather than a terminal record.
- `settlement`: balances are computed from current revisions only, and the response signals when a prior plan may have been invalidated.

## Impact

- **New API surface**: `PATCH /groups/{id}/expenses/{expenseId}`, `POST /groups/{id}/expenses/{expenseId}/void`, `GET /groups/{id}/expenses/{expenseId}/revisions`.
- **Schema**: `expenses` gains `revision`, `superseded_by`, and `voided_at`.
- **Open question blocking design**: should a void be permitted after a settlement plan has been acted on, given that Splitr has no idea whether anyone actually paid? This needs a product decision before the design can be written.
