## 1. Schema and persistence

- [ ] 1.1 Create `groups` table (id, name, created_at) with a non-empty name constraint.
- [ ] 1.2 Create `group_members` table (id, group_id, name) with a unique constraint on (group_id, trimmed name).
- [ ] 1.3 Create `expenses` table (id, group_id, description, amount_minor BIGINT, payer_member_id, created_at) with a positive-amount check constraint.
- [ ] 1.4 Create `expense_participants` join table (expense_id, member_id, share_minor) with a foreign key to `group_members`.
- [ ] 1.5 Add a constraint or insert-time check that payer and participants belong to the expense's group.

## 2. Group management capability

- [ ] 2.1 `POST /groups`: create a group with name and members.
- [ ] 2.2 Reject empty or whitespace-only group names.
- [ ] 2.3 Reject groups with fewer than two members.
- [ ] 2.4 Reject duplicate member names after trimming whitespace.
- [ ] 2.5 `GET /groups/{id}`: return name and member list; 404 for unknown id.
- [ ] 2.6 Confirm no route exists to add or remove members after creation.

## 3. Expense recording capability

- [ ] 3.1 `POST /groups/{id}/expenses`: record description, amount, payer, participants.
- [ ] 3.2 Validate amount is a positive integer; reject zero, negative, and fractional.
- [ ] 3.3 Validate payer and every participant belong to the group; reject empty participant lists.
- [ ] 3.4 Implement even split with remainder assigned in ascending member-identifier order.
- [ ] 3.5 Persist computed per-participant shares alongside the expense.
- [ ] 3.6 `GET /groups/{id}/expenses`: list ordered by creation time; empty list, not error, when none.

## 4. Settlement capability

- [ ] 4.1 Compute net balance per member as total paid minus total shares.
- [ ] 4.2 `GET /groups/{id}/balances`: return every member including those with a zero balance.
- [ ] 4.3 Implement greedy largest-creditor / largest-debtor settlement per design.md.
- [ ] 4.4 `GET /groups/{id}/settlement`: return the transfer list; empty list when nothing to settle.
- [ ] 4.5 Confirm settlement requests are side-effect free and repeatable.

## 5. Verification against specs

- [ ] 5.1 Test: shares sum exactly to the expense amount, across a range of amounts and participant counts.
- [ ] 5.2 Test: 1000 minor units across 3 participants yields 334 / 333 / 333.
- [ ] 5.3 Test: balances sum to exactly zero for any group.
- [ ] 5.4 Test: applying every transfer in a plan brings all balances to zero.
- [ ] 5.5 Test: the two-creditor example from the settlement spec produces exactly two transfers.
- [ ] 5.6 Confirm the greedy-vs-minimal divergence flagged in design.md is either accepted or raised as a follow-up change before archiving.
