## Why

People sharing costs (housemates, a trip, a dinner) currently track who paid what in a group chat or a spreadsheet, then argue about the arithmetic. The counting is trivial but nobody wants to own it, and partial payments make running totals go stale immediately.

Splitr needs a foundation that answers one question reliably: given everything the group has recorded, who owes whom, and how few transfers settle it? Everything else Splitr might do later (currencies, receipts, reminders) builds on that answer being correct.

## What Changes

- Introduce groups as the unit of shared spending, with a fixed member list.
- Allow recording an expense: an amount, who paid it, and which members it is split across.
- Support even splits only. A £30 dinner across 3 members is £10 each.
- Compute each member's net balance across all of a group's expenses.
- Compute a settlement plan: the set of transfers that clears all balances, minimizing the number of transfers.
- Expose all of the above over the HTTP API. No UI, no auth, no payment execution.

Non-goals for this change: uneven or percentage splits, editing or deleting a recorded expense, more than one currency, and any notion of a transfer actually happening.

## Capabilities

### New Capabilities

- `group-management`: creating a group, and the membership list that expenses are recorded against.
- `expense-recording`: recording an expense against a group, including who paid and how it divides across members.
- `settlement`: deriving per-member net balances from recorded expenses, and producing a minimal set of transfers that clears them.

### Modified Capabilities

None. This is the first change; no specs exist yet.

## Impact

- **New API surface**: `POST /groups`, `GET /groups/{id}`, `POST /groups/{id}/expenses`, `GET /groups/{id}/balances`, `GET /groups/{id}/settlement`.
- **New persistence**: `groups`, `group_members`, and `expenses` tables. Amounts stored as integer minor units, per project convention.
- **No changes to existing systems.** Nothing exists yet.
- **Forward risk**: the settlement capability assumes a single currency throughout. That assumption is deliberate and will have to be revisited, which is expected to be the next change.
