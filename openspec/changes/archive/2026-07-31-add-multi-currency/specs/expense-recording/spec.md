## MODIFIED Requirements

### Requirement: Recording an expense

The system SHALL allow a client to record an expense against a group, specifying a description, an amount in integer minor units, **the currency of that amount**, the member who paid, and the members the expense is split across. The system SHALL assign each expense a unique identifier and record the time it was created. When the client omits a currency, the system SHALL record the expense in the group's default currency.

#### Scenario: Expense recorded against the whole group

- **WHEN** a client records an expense of 3000 minor units in EUR paid by "Ana" and split across "Ana", "Ben", and "Cara"
- **THEN** the system stores the expense with currency EUR and returns its identifier
- **AND** the expense is included in that group's EUR balance calculation

#### Scenario: Expense split across a subset of members

- **WHEN** a client records an expense paid by "Ana" and split across only "Ana" and "Ben"
- **THEN** the system stores the expense with a participant list of exactly those two members
- **AND** "Cara" is unaffected by that expense

#### Scenario: Currency omitted falls back to the group default

- **WHEN** a client records an expense without a currency in a group whose default currency is GBP
- **THEN** the system stores the expense with currency GBP

## ADDED Requirements

### Requirement: Expense currency validity

The system SHALL require an expense currency to be a three-letter ISO 4217 alphabetic code. The system SHALL reject unknown codes, and SHALL treat currency codes case-insensitively, normalizing them to uppercase before storage.

#### Scenario: Valid currency accepted

- **WHEN** a client records an expense with currency "EUR"
- **THEN** the system stores the expense with currency EUR

#### Scenario: Lowercase code normalized

- **WHEN** a client records an expense with currency "eur"
- **THEN** the system stores the expense with currency EUR

#### Scenario: Unknown code rejected

- **WHEN** a client records an expense with currency "XYZ"
- **THEN** the system rejects the request with a validation error naming the unknown code
- **AND** no expense is recorded

#### Scenario: Malformed code rejected

- **WHEN** a client records an expense with a currency that is not three alphabetic characters
- **THEN** the system rejects the request with a validation error
- **AND** no expense is recorded

### Requirement: Splitting is per expense, within one currency

The system SHALL split each expense only among its own participants, in that expense's currency. The remainder-assignment rule SHALL operate independently per expense, so that the shares of an expense sum exactly to that expense's amount in its own currency.

#### Scenario: Shares reconcile within a currency

- **WHEN** an expense of 1000 minor units in EUR is split across 3 participants
- **THEN** the EUR shares are 334, 333, and 333 minor units
- **AND** the shares sum to exactly 1000 EUR

#### Scenario: Two currencies do not interact when splitting

- **WHEN** a group has one EUR expense and one GBP expense
- **THEN** each expense's shares are computed against its own amount and currency only
- **AND** neither expense's remainder assignment affects the other
