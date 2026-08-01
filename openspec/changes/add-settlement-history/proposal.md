## Why

Splitr's settlement plans are recomputed on every request and never recorded, per the "Settlement does not move money" requirement. That is the right boundary, but it leaves a gap: a group settles up in March, records more expenses in April, and has no way to see that March was ever cleared. Every plan looks like it covers all of history.

Recording that a group *declared* itself settled is different from Splitr executing a transfer. Splitr can honestly assert the first without touching the second.

## What Changes

- Allow a group to record a settlement checkpoint: a client-asserted marker that balances up to a point in time were considered cleared.
- Compute balances and plans from expenses recorded *after* the most recent checkpoint.
- Expose checkpoint history, so a group can see when it last settled.
- Checkpoints are append-only. They cannot be edited or deleted, only superseded by a later one.

Explicit non-goal: Splitr still does not verify that money moved. A checkpoint is a claim by the group, recorded as such.

## Capabilities

### New Capabilities

- `settlement-checkpoints`: recording, listing, and superseding the markers that divide a group's expense history into settled periods.

### Modified Capabilities

- `settlement`: balances and plans are scoped to expenses after the latest checkpoint rather than all of history.

## Impact

- **New API surface**: `POST /groups/{id}/checkpoints`, `GET /groups/{id}/checkpoints`.
- **Schema**: new `settlement_checkpoints` table (group_id, created_at, note).
- **Behavioral change to existing endpoints**: `GET /groups/{id}/balances` and `/settlement` change meaning for any group with a checkpoint. Not a payload-shape break, which arguably makes it more dangerous rather than less.
