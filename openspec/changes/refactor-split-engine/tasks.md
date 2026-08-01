## 1. Characterization

- [x] 1.1 Generate split outputs across amounts 1..10000 and participant counts 2..12 from the current code.
- [x] 1.2 Commit the generated fixture as the refactor's reference output.
- [x] 1.3 Confirm the fixture captures which participant receives each extra minor unit, not just the sum.

## 2. Extract the split engine

- [x] 2.1 Create `split-engine` with `splitEvenly(amountMinor, participantIds[])`.
- [x] 2.2 Move remainder assignment (ascending participant identifier) into the module unchanged.
- [ ] 2.3 Assert the module reproduces the characterization fixture byte for byte.
- [ ] 2.4 Replace the inlined logic in the expense-recording handler with a call.

## 3. Extract the settlement matcher

- [ ] 3.1 Move the greedy largest-creditor / largest-debtor matcher into the same module.
- [ ] 3.2 Keep per-currency partitioning in the caller, not the module.
- [ ] 3.3 Replace the inlined matcher in the settlement handler with a call.

## 4. Tests

- [ ] 4.1 Convert split-arithmetic endpoint tests into direct unit tests.
- [ ] 4.2 Keep one endpoint-level integration test per capability.
- [ ] 4.3 Confirm every scenario in `expense-recording` and `settlement` still passes unmodified.

## 5. Confirm no behavior changed

- [ ] 5.1 Re-run the full suite against the pre-refactor fixture.
- [ ] 5.2 Confirm no spec file was touched by this change, as `skip_specs: true` asserts.
