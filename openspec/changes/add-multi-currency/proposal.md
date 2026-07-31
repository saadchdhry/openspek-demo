## Why

Splitr's first slice assumed every amount in a group is the same currency. That assumption survives exactly until the first trip abroad: someone pays for the hotel in EUR, someone else pays for dinner in GBP, and the balances are now nonsense added together as bare integers.

The existing `settlement` capability states this assumption as a requirement rather than leaving it implicit, which means it can be changed honestly instead of quietly reinterpreted.

## What Changes

- Every expense carries an explicit ISO 4217 currency code. **BREAKING**: the expense payload gains a required `currency` field.
- Balances are reported per currency rather than as a single figure per member.
- Settlement produces one independent plan per currency. Splitr does not convert between currencies.
- **BREAKING**: `GET /groups/{id}/balances` and `GET /groups/{id}/settlement` change shape, from a flat list to a list grouped by currency.
- Groups declare a default currency at creation, used when an expense omits one.

Non-goals: exchange rates, conversion, and any attempt to net a EUR debt against a GBP credit. Splitr reports what is owed in the currency it was spent in. Deciding what a EUR debt is "worth" in GBP is a judgement about a rate on a date, and Splitr has no defensible basis for making it.

## Capabilities

### New Capabilities

None. This change adds no new capability; it changes the behavior of two existing ones.

### Modified Capabilities

- `settlement`: the "Single currency across a group" requirement is replaced. Balances become per-currency, and the settlement plan requirement is restated to produce one plan per currency rather than one plan overall.
- `expense-recording`: recording an expense now accepts and requires a currency, and rejects unknown codes.
- `group-management`: group creation accepts a default currency.

## Impact

- **Breaking API changes**: `POST /groups/{id}/expenses` requires `currency`. `GET /groups/{id}/balances` and `GET /groups/{id}/settlement` change response shape. There are no external consumers yet, so no versioning strategy is needed. That will not be true after this ships.
- **Schema migration**: `expenses` gains a `currency CHAR(3)` column; `groups` gains `default_currency CHAR(3)`. Existing rows need backfilling, which is trivial today because there are none in production.
- **Settlement logic**: partitions by currency before running the existing greedy algorithm. The algorithm itself is unchanged, it just runs N times.
- **Resolves** the forward risk recorded in the `add-expense-splitting` proposal, which called this out as the expected next change.
