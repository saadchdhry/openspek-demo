## MODIFIED Requirements

### Requirement: Net balance per member

The system SHALL compute each group member's net balance **per currency** as the total they paid in that currency minus the total of their shares in that currency, across **the expenses recorded after the group's most recent settlement checkpoint**. When a group has no checkpoint, the system SHALL consider all of its expenses. A positive balance means the member is owed money in that currency; a negative balance means the member owes money in that currency. The system SHALL NOT combine balances across currencies.

#### Scenario: Single expense

- **WHEN** "Ana" pays 3000 minor units in EUR split evenly across "Ana", "Ben", and "Cara"
- **THEN** Ana's EUR balance is +2000
- **AND** Ben's EUR balance is -1000
- **AND** Cara's EUR balance is -1000

#### Scenario: Member with no involvement

- **WHEN** a member has neither paid for nor participated in any expense
- **THEN** that member has no non-zero balance in any currency

#### Scenario: Balances always sum to zero

- **WHEN** balances are computed for any group
- **THEN** for each currency independently, the sum of all member balances in that currency is exactly 0

#### Scenario: Empty group

- **WHEN** balances are requested for a group with no recorded expenses
- **THEN** the system returns an empty set of currency balances

#### Scenario: Balances in two currencies are reported separately

- **WHEN** a group has expenses in both EUR and GBP
- **THEN** each member's balance is reported once per currency
- **AND** no single figure combining EUR and GBP is reported for any member

#### Scenario: A member owed in one currency and owing in another

- **WHEN** Ana's EUR balance is +5000 and her GBP balance is -3000
- **THEN** both balances are reported unchanged
- **AND** the system does not net them against each other

#### Scenario: Expenses before the latest checkpoint are excluded

- **WHEN** a group records expenses, then a checkpoint, then one further expense of 900 EUR paid by Ana and split across three members
- **THEN** balances reflect only the 900 EUR expense
- **AND** the expenses preceding the checkpoint do not appear in any balance

#### Scenario: Group settled with nothing since

- **WHEN** a group's most recent checkpoint is later than every one of its expenses
- **THEN** every member's balance is 0 in every currency

#### Scenario: Only the latest checkpoint applies

- **WHEN** a group has three checkpoints
- **THEN** balances are computed from expenses after the newest checkpoint only
- **AND** the two earlier checkpoints do not affect the result
