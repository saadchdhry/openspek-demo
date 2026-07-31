## Why

The even-split arithmetic and the remainder-assignment rule are currently inlined in the expense-recording HTTP handler. `add-multi-currency` added a currency dimension to the same function, and `add-expense-editing` will need to re-run the split when an expense is revised. Three callers, one of them not written yet, all reaching into a request handler.

This is a code-shape problem, not a behavior problem. Splitr computes exactly the same numbers before and after.

## What Changes

- Extract split arithmetic into a standalone module with no HTTP or database dependency.
- Extract the greedy settlement matcher alongside it.
- Convert the existing endpoint tests that cover split arithmetic into direct unit tests of the module, keeping the endpoint tests as thin integration checks.
- No API change. No schema change. No observable behavior change.

## Capabilities

### New Capabilities

None. This change introduces no behavior.

### Modified Capabilities

None. Every requirement in `expense-recording` and `settlement` holds exactly as written, before and after.

This change sets `skip_specs: true` in its `.openspec.yaml`. That is the correct marker for a pure refactor: specs describe behavior, behavior does not change, therefore no spec changes. Inventing a requirement here to satisfy validation would be worse than useless.

## Impact

- **Code**: new `split-engine` module. `expense-recording` and `settlement` handlers become callers.
- **Tests**: split-arithmetic tests move from endpoint level to unit level. Net increase in coverage at no behavioral cost.
- **No API, schema, or dependency changes.**
- **Unblocks** `add-expense-editing`, which needs to re-run a split outside the create path.
