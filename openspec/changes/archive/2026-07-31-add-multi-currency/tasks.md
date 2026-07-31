## 1. Schema migration

- [x] 1.1 Add `groups.default_currency CHAR(3)` nullable.
- [x] 1.2 Add `expenses.currency CHAR(3)` nullable.
- [x] 1.3 Choose the backfill constant and record it in the migration runbook.
- [x] 1.4 Backfill `groups.default_currency`, then `expenses.currency` from the owning group.
- [x] 1.5 Set both columns `NOT NULL`.
- [x] 1.6 Verify rollback: dropping both columns leaves the previous release working.

## 2. Currency validation

- [x] 2.1 Add an ISO 4217 alphabetic-code allowlist.
- [x] 2.2 Normalize codes to uppercase before storage.
- [x] 2.3 Reject unknown codes with an error naming the code.
- [x] 2.4 Reject codes that are not three alphabetic characters.

## 3. Group management changes

- [x] 3.1 `POST /groups` accepts and requires `defaultCurrency`; validate it.
- [x] 3.2 `GET /groups/{id}` returns `defaultCurrency`.
- [x] 3.3 Confirm no route can change `defaultCurrency` after creation.

## 4. Expense recording changes

- [x] 4.1 `POST /groups/{id}/expenses` accepts `currency`.
- [x] 4.2 Resolve an omitted currency to the group default at write time, and store the resolved value.
- [x] 4.3 Confirm split arithmetic runs per expense and is unaffected by other currencies.

## 5. Settlement changes

- [x] 5.1 Partition balances by currency.
- [x] 5.2 Return balances grouped by currency; never combine across currencies.
- [x] 5.3 Run the existing greedy matcher once per currency, unchanged.
- [x] 5.4 Return one plan per currency, each transfer naming its currency.
- [x] 5.5 Return an empty list of plans when nothing is owed in any currency.
- [x] 5.6 Reject `convertTo` or an exchange-rate parameter with a validation error rather than ignoring it.

## 6. Verification against specs

- [x] 6.1 Test: a group with EUR and GBP expenses reports two separate balance sets.
- [x] 6.2 Test: a member +5000 EUR and -3000 GBP has both reported, unnetted.
- [x] 6.3 Test: balances sum to zero within each currency independently.
- [x] 6.4 Test: applying all transfers in all plans zeroes every balance in every currency.
- [x] 6.5 Test: no transfer ever names two currencies.
- [x] 6.6 Test: an expense with currency omitted takes the group default.
- [x] 6.7 Test: "eur" normalizes to EUR; "XYZ" and "EURO" are rejected.
- [x] 6.8 Confirm the JPY minor-unit gap in design.md is raised as a follow-up change before archiving.
