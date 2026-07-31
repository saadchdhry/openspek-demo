# settlement Specification

## Purpose
Turns a group's recorded expenses into the answer people actually want: each member's net position, and the shortest list of transfers that clears everyone out.
## Requirements
### Requirement: Net balance per member

The system SHALL compute each group member's net balance as the total they paid minus the total of their shares across all expenses in the group. A positive balance means the member is owed money; a negative balance means the member owes money.

#### Scenario: Single expense

- **WHEN** "Ana" pays 3000 minor units split evenly across "Ana", "Ben", and "Cara"
- **THEN** Ana's balance is +2000
- **AND** Ben's balance is -1000
- **AND** Cara's balance is -1000

#### Scenario: Member with no involvement

- **WHEN** a member has neither paid for nor participated in any expense
- **THEN** that member's balance is 0

#### Scenario: Balances always sum to zero

- **WHEN** balances are computed for any group
- **THEN** the sum of all member balances is exactly 0

#### Scenario: Empty group

- **WHEN** balances are requested for a group with no recorded expenses
- **THEN** every member's balance is 0

### Requirement: Single currency across a group

The system SHALL treat all amounts within a group as the same, unspecified currency. Balances and settlement amounts SHALL be reported as bare integer minor units with no currency attached.

#### Scenario: Amounts are comparable without conversion

- **WHEN** balances are computed across any number of expenses in a group
- **THEN** amounts are summed directly with no conversion step

#### Scenario: No currency is recorded

- **WHEN** a client records an expense
- **THEN** the system does not accept or store a currency for it

### Requirement: Settlement plan

The system SHALL produce a settlement plan: a list of transfers, each naming a payer, a recipient, and an amount, that brings every member's balance to zero. The system SHALL minimize the number of transfers in the plan.

#### Scenario: Simple two-party settlement

- **WHEN** Ana's balance is +1000 and Ben's is -1000
- **THEN** the plan contains exactly one transfer of 1000 from Ben to Ana

#### Scenario: Minimal transfers preferred

- **WHEN** Ana is +2000, Ben is -1000, and Cara is -1000
- **THEN** the plan contains exactly two transfers, one from each debtor to Ana
- **AND** no plan with fewer transfers exists

#### Scenario: Plan clears all balances

- **WHEN** a settlement plan is produced for any group
- **THEN** applying every transfer in the plan brings all member balances to 0

#### Scenario: Nothing to settle

- **WHEN** every member's balance is already 0
- **THEN** the plan is an empty list

### Requirement: Settlement does not move money

The system SHALL treat a settlement plan as advisory output only. The system SHALL NOT initiate, execute, or record the completion of any transfer.

#### Scenario: Plan is recomputed, never stored as settled

- **WHEN** a client requests a group's settlement plan twice with no expenses recorded in between
- **THEN** the system returns the same plan both times
- **AND** no state changed as a result of either request

#### Scenario: No payment integration is exposed

- **WHEN** a client attempts to mark a transfer as paid or to execute one
- **THEN** the system responds with a not-found error

