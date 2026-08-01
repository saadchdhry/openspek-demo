## Context

See proposal.md ("Why"). Settlement currently reads every expense in a group. This change introduces a boundary marker that scopes that read.

## Goals / Non-Goals

**Goals**

- A checkpoint is a timestamp and nothing more. No amounts are copied into it.
- Balances remain fully derived. Adding checkpoints must not introduce a stored aggregate.
- The existing per-currency partitioning and greedy matcher are untouched.

**Non-Goals**

- Verifying payment. Splitr has no bank connection and will not pretend otherwise.
- Per-member checkpoints. A checkpoint applies to the whole group.
- Retroactively inserting a checkpoint at a past time.

## Decisions

### A checkpoint stores a timestamp, not a snapshot

`settlement_checkpoints(id, group_id, created_at, note)`. Balances are computed with `WHERE expenses.created_at > (SELECT MAX(created_at) ...)`.

*Alternative considered:* snapshot each member's balance into the checkpoint row. Rejected: it is a stored aggregate, which is exactly what the original design decided against. It would also go stale the moment `add-expense-editing` lands and an expense predating the checkpoint is corrected.

### Checkpoints are append-only, with no retroactive insertion

`created_at` is server-assigned, never client-supplied.

*Alternative considered:* let clients backdate a checkpoint to when they actually settled. Rejected for now: a backdated checkpoint can silently re-open or close periods around expenses recorded in between, and there is no obvious correct behavior. Worth revisiting as its own change.

### Interaction with `add-expense-editing` is unresolved

That change is still at proposal stage and proposes revisions to expenses. An edit to an expense that predates the latest checkpoint changes a settled period.

Neither change can decide this alone. Whichever archives second must state the rule. Flagged here so it is not discovered at implementation time.

## Risks / Trade-offs

- **The meaning of `GET /balances` changes without its payload shape changing.** → A caller upgrading gets different numbers with no schema break to alert them. Mitigation: include the active checkpoint (or null) in the balances response, so the change in meaning is visible in the payload.
- **Clock skew on `created_at` could place an expense on the wrong side of a checkpoint.** → Both timestamps come from the database, not application servers. Ties are resolved as "after the checkpoint", so a same-instant expense stays outstanding rather than being silently cleared.
- **A group could checkpoint while balances are non-zero**, declaring itself settled when it is not. → Accepted. Splitr records claims; it does not audit them. The API should not block it.

## Migration Plan

Additive. New table, no changes to existing rows. Groups without checkpoints behave exactly as before.

Rollback: drop the table. Balance computation falls back to all-of-history, which is the current behavior.

## Open Questions

- Should the balances response include the count of expenses excluded by the checkpoint? Deferrable: useful for debugging, changes no requirement.
