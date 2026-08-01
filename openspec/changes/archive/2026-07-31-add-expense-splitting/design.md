## Context

Splitr has no existing code. See proposal.md ("Why") for motivation. This design covers the first vertical slice: schema, split arithmetic, and settlement.

The only genuinely non-obvious part is settlement. Everything else is CRUD.

## Goals / Non-Goals

**Goals**

- Balances are derived, never stored, so they can never drift from the expenses that produced them.
- Split arithmetic is exact: shares always sum to the expense amount, with no rounding drift.
- Settlement is deterministic. The same inputs always produce the same plan, so the output is testable.

**Non-Goals**

- Optimizing settlement for anything other than transfer count (for example, preferring transfers between people who already owe each other).
- Concurrency control beyond ordinary transactional inserts. Groups are small and trusted.
- Any caching layer. Recomputing from expenses is cheap at this size.

## Decisions

### Store expenses, derive balances

Balances and settlement plans are computed on read from the `expenses` and `expense_participants` tables. Nothing aggregated is persisted.

*Alternative considered:* maintain a running `balance` column per member, updated on each insert. Rejected: it introduces a second source of truth that can drift from the expense log, and the read cost it saves is irrelevant for groups of fewer than 20 people.

### Integer minor units, with remainder assigned by member identifier

All amounts are `BIGINT` minor units. An expense of amount `A` across `n` participants gives each `floor(A/n)`, then the first `A mod n` participants in ascending member-identifier order receive one extra minor unit.

*Alternative considered:* distribute the remainder to the payer, or at random. Rejected: ordering by member identifier is deterministic and therefore testable, which the "shares always reconcile" scenario in `expense-recording` depends on. Random or payer-based assignment makes the fairness question ("why did Ana always get the extra penny?") a product decision, and this change does not need to answer it.

*Alternative considered:* decimal or floating-point amounts. Rejected outright by project convention, and floats cannot satisfy the exact-sum requirement.

### Greedy largest-creditor / largest-debtor settlement

Sort creditors and debtors by absolute balance. Repeatedly match the largest creditor against the largest debtor, transferring the smaller of the two magnitudes, until all balances are zero.

This yields at most `n-1` transfers for `n` members with non-zero balances.

*Alternative considered:* exact minimum-transfer settlement, which is the partition problem and NP-hard. Rejected: exponential cost for a result that differs from greedy only in constructed cases. This is a real gap between the spec and the implementation and is recorded under Risks below.

### One spec directory per capability, three tables

`groups`, `group_members`, `expenses`, `expense_participants`. The participant list is a join table rather than a JSON column so that "participant must belong to the group" is a foreign-key constraint rather than application logic.

## Risks / Trade-offs

- **Greedy settlement is not provably minimal, but the spec says "minimize the number of transfers."** → The greedy result is optimal for the common cases and never worse than `n-1` transfers. This is a known divergence: either the implementation gets an exact solver later, or the requirement gets softened to "at most one fewer transfer than there are members with non-zero balances." Flagged for the reviewer now rather than discovered at test time.
- **Fixed membership is a real product limitation.** → Accepted deliberately for this change. Adding a member later means creating a new group, which is bad. It is scoped out to keep the first slice honest, and it will need its own change.
- **Single currency is baked into settlement.** → Accepted and explicitly written into the settlement spec, so that changing it later shows up as a `MODIFIED` requirement rather than a silent reinterpretation.
- **No expense editing or deletion.** → A mistyped expense is uncorrectable. Acceptable for a first slice; correcting it is a follow-up change.

## Migration Plan

None. Greenfield, no existing data, no deployed consumers.

## Open Questions

- Should a settlement plan round-trip to a stable transfer identifier so a future change can mark transfers as paid? Deferrable: it does not change any requirement in this change, and the settlement spec already states that plans are advisory and recomputed.
