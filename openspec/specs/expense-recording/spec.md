# expense-recording Specification

## Purpose
Defines how a shared cost enters the system: who paid, how much, and which members the cost divides across. Recorded expenses are the only input from which balances and settlements are derived.
## Requirements
### Requirement: Recording an expense

The system SHALL allow a client to record an expense against a group, specifying a description, an amount in integer minor units, the member who paid, and the members the expense is split across. The system SHALL assign each expense a unique identifier and record the time it was created.

#### Scenario: Expense recorded against the whole group

- **WHEN** a client records an expense of 3000 minor units paid by "Ana" and split across "Ana", "Ben", and "Cara"
- **THEN** the system stores the expense and returns its identifier
- **AND** the expense is included in that group's balance calculation

#### Scenario: Expense split across a subset of members

- **WHEN** a client records an expense paid by "Ana" and split across only "Ana" and "Ben"
- **THEN** the system stores the expense with a participant list of exactly those two members
- **AND** "Cara" is unaffected by that expense

### Requirement: Expense amount validity

The system SHALL require an expense amount to be a positive integer in minor units. The system SHALL reject zero, negative, and non-integer amounts.

#### Scenario: Zero amount rejected

- **WHEN** a client records an expense with an amount of 0
- **THEN** the system rejects the request with a validation error
- **AND** no expense is recorded

#### Scenario: Negative amount rejected

- **WHEN** a client records an expense with an amount below 0
- **THEN** the system rejects the request with a validation error
- **AND** no expense is recorded

#### Scenario: Fractional amount rejected

- **WHEN** a client records an expense with an amount of 10.5
- **THEN** the system rejects the request with a validation error stating amounts are integer minor units

### Requirement: Participants must belong to the group

The system SHALL require that the paying member and every participant belong to the group the expense is recorded against, and that the participant list contains at least one member.

#### Scenario: Payer outside the group

- **WHEN** a client records an expense whose payer is not a member of the group
- **THEN** the system rejects the request with a validation error
- **AND** no expense is recorded

#### Scenario: Participant outside the group

- **WHEN** a client records an expense listing a participant who is not a member of the group
- **THEN** the system rejects the request with a validation error
- **AND** no expense is recorded

#### Scenario: Empty participant list

- **WHEN** a client records an expense with no participants
- **THEN** the system rejects the request with a validation error

### Requirement: Even split with deterministic remainder

The system SHALL divide an expense evenly across its participants. When the amount does not divide evenly, the system SHALL distribute the remaining minor units one each to participants in ascending order of member identifier, so that the shares always sum exactly to the expense amount.

#### Scenario: Amount divides evenly

- **WHEN** an expense of 3000 minor units is split across 3 participants
- **THEN** each participant's share is 1000 minor units

#### Scenario: Amount does not divide evenly

- **WHEN** an expense of 1000 minor units is split across 3 participants
- **THEN** the shares are 334, 333, and 333 minor units
- **AND** the extra minor unit goes to the participant with the lowest member identifier
- **AND** the shares sum to exactly 1000

#### Scenario: Shares always reconcile

- **WHEN** any expense is split across any number of participants
- **THEN** the sum of all participant shares equals the expense amount exactly

### Requirement: Listing a group's expenses

The system SHALL allow a client to list all expenses recorded against a group, returning for each its identifier, description, amount, payer, participants, and creation time, ordered by creation time.

#### Scenario: Expenses listed

- **WHEN** a client lists the expenses of a group with three recorded expenses
- **THEN** the system returns all three, ordered by creation time

#### Scenario: Group with no expenses

- **WHEN** a client lists the expenses of a group that has none
- **THEN** the system returns an empty list rather than an error

