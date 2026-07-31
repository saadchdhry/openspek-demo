## 1. Schema

- [x] 1.1 Create `settlement_checkpoints` table (id, group_id, created_at, note).
- [x] 1.2 Server-assign `created_at`; reject any client-supplied value.
- [x] 1.3 Index on (group_id, created_at DESC) for the latest-checkpoint lookup.

## 2. Checkpoint capability

- [x] 2.1 `POST /groups/{id}/checkpoints` with an optional note.
- [x] 2.2 `GET /groups/{id}/checkpoints` in reverse chronological order.
- [ ] 2.3 Confirm no route can edit or delete a checkpoint.
- [ ] 2.4 Ensure responses describe checkpoints as group-asserted, never as verified payment.

## 3. Settlement scoping

- [ ] 3.1 Resolve the latest checkpoint per group.
- [ ] 3.2 Scope balance computation to expenses after it; fall back to all history when absent.
- [ ] 3.3 Resolve same-instant ties as "after the checkpoint".
- [ ] 3.4 Include the active checkpoint (or null) in the balances response, per design.md Risks.

## 4. Verification against specs

- [ ] 4.1 Test: expenses before the latest checkpoint are excluded from balances.
- [ ] 4.2 Test: a group checkpointed after its last expense shows all-zero balances.
- [ ] 4.3 Test: with three checkpoints, only the newest applies.
- [ ] 4.4 Test: a group with no checkpoint behaves exactly as before this change.
- [ ] 4.5 Test: checkpoint edit and delete both refused.

## 5. Cross-change coordination

- [ ] 5.1 **Blocked.** Resolve the interaction with `add-expense-editing`: what happens when an expense predating a checkpoint is revised. Neither change can decide this alone. Must be settled before whichever of the two archives second.
