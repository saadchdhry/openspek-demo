## MODIFIED Requirements

### Requirement: Net balance per member

The system SHALL compute each group member's net balance **per currency** as the total they paid in that currency minus the total of their shares in that currency, across all expenses in the group. A positive balance means the member is owed money in that currency; a negative balance means the member owes money in that currency. The system SHALL NOT combine balances across currencies.

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

### Requirement: Settlement plan

The system SHALL produce one settlement plan **per currency** present in the group: a list of transfers, each naming a payer, a recipient, an amount, and that currency, that brings every member's balance in that currency to zero. The system SHALL minimize the number of transfers within each per-currency plan. The system SHALL NOT produce transfers that span two currencies.

#### Scenario: Simple two-party settlement

- **WHEN** Ana's EUR balance is +1000 and Ben's is -1000
- **THEN** the EUR plan contains exactly one transfer of 1000 EUR from Ben to Ana

#### Scenario: Minimal transfers preferred

- **WHEN** in EUR, Ana is +2000, Ben is -1000, and Cara is -1000
- **THEN** the EUR plan contains exactly two transfers, one from each debtor to Ana
- **AND** no EUR plan with fewer transfers exists

#### Scenario: Plan clears all balances

- **WHEN** settlement plans are produced for any group
- **THEN** applying every transfer in every plan brings all member balances in all currencies to 0

#### Scenario: Nothing to settle

- **WHEN** every member's balance in every currency is already 0
- **THEN** the system returns an empty list of plans

#### Scenario: Currencies settle independently

- **WHEN** a group has non-zero balances in both EUR and GBP
- **THEN** the system returns one EUR plan and one GBP plan
- **AND** every transfer names exactly one currency
- **AND** no transfer converts between currencies

## REMOVED Requirements

### Requirement: Single currency across a group

**Reason**: Directly contradicted by this change. Amounts within a group are no longer assumed to share a currency, and both balances and settlement now carry an explicit currency.

**Migration**: Replaced by the per-currency behavior in the modified "Net balance per member" and "Settlement plan" requirements above, and by the "Expense currency" requirement in the `expense-recording` capability. Existing single-currency groups behave identically once their expenses are backfilled with the group's default currency.

## ADDED Requirements

### Requirement: No currency conversion

The system SHALL NOT convert between currencies, apply exchange rates, or report a total across currencies. Currency conversion is outside the boundary of what Splitr asserts.

#### Scenario: No conversion is offered

- **WHEN** a client requests balances or a settlement plan for a multi-currency group
- **THEN** the response contains no converted, combined, or estimated total
- **AND** the response contains no exchange rate

#### Scenario: Conversion is not accepted as input

- **WHEN** a client supplies an exchange rate or a target currency when requesting balances or settlement
- **THEN** the system rejects the request with a validation error
