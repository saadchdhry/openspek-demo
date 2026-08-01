## Context

See proposal.md. Split arithmetic lives inside the `POST /groups/{id}/expenses` handler and is about to acquire a third caller.

## Goals / Non-Goals

**Goals**

- The extracted module is pure: amounts and participant identifiers in, shares out. No I/O, no clock, no database.
- Behavior is provably identical. The existing spec scenarios are the acceptance criteria, unchanged.

**Non-Goals**

- Changing the remainder rule, the matcher, or any rounding behavior. If this change alters a single computed number, it has failed.
- Optimizing the greedy matcher. That divergence from "minimize transfers" is still open from `add-expense-splitting` and is deliberately not touched here.

## Decisions

### Pure module, identifiers in and out

`splitEvenly(amountMinor, participantIds[]) → Map<participantId, shareMinor>`. The caller supplies already-sorted identifiers; the module does not query anything.

*Alternative considered:* pass the whole expense entity. Rejected: it drags the persistence model into a module whose entire value is having no dependencies.

### Currency is the caller's concern

The module never sees a currency code. Callers partition by currency first, then call in. This matches the specs, which state that splitting operates within one currency.

*Alternative considered:* pass the currency for minor-unit precision. Rejected: precision handling does not exist yet (the JPY gap from `add-multi-currency` is still open). Adding a parameter the module ignores would imply a guarantee that is not there.

### Verify by characterization before refactoring

Capture the current output over a generated range of amounts and participant counts, refactor, then assert byte-identical output. The specs say shares must always reconcile; this proves the refactor preserves *which* participant got the extra minor unit, which is stated in the spec but easy to break silently.

## Risks / Trade-offs

- **A silent change to remainder assignment would satisfy every existing test** (they mostly assert the sum) **while changing who pays the extra penny.** → This is the whole reason for the characterization step above. It is the one real risk in an otherwise mechanical change.
- **Merge conflict with `add-expense-editing`**, which touches the same handler. → Land this first. It is small, and it is what that change wanted anyway.

## Migration Plan

None. Internal refactor, no deployed surface changes. Rollback is a revert.

## Open Questions

None.
