# group-management Specification

## Purpose
Defines the group: the named container that a fixed set of members records shared expenses against. Every expense and every balance in Splitr is scoped to exactly one group.
## Requirements
### Requirement: Group creation

The system SHALL allow a client to create a group with a display name, an initial list of members, and **a default currency**. The system SHALL assign each group a unique identifier and each member a unique identifier scoped to that group. The default currency SHALL be a three-letter ISO 4217 alphabetic code, and SHALL be used for any expense recorded without an explicit currency.

#### Scenario: Group created with members

- **WHEN** a client creates a group named "Lisbon trip" with members "Ana", "Ben", and "Cara" and default currency EUR
- **THEN** the system returns a group identifier and three member identifiers
- **AND** the group's member list contains exactly those three members
- **AND** the group's default currency is EUR

#### Scenario: Group name is required

- **WHEN** a client creates a group with an empty or whitespace-only name
- **THEN** the system rejects the request with a validation error
- **AND** no group is created

#### Scenario: Group requires at least two members

- **WHEN** a client creates a group with fewer than two members
- **THEN** the system rejects the request with a validation error
- **AND** no group is created

#### Scenario: Invalid default currency rejected

- **WHEN** a client creates a group with a default currency that is not a known ISO 4217 code
- **THEN** the system rejects the request with a validation error
- **AND** no group is created

### Requirement: Member name uniqueness within a group

The system SHALL reject a group whose member names are not unique after trimming surrounding whitespace, so that a member can be identified unambiguously by name within a group.

#### Scenario: Duplicate member names rejected

- **WHEN** a client creates a group with members "Ana", "Ben", and "Ana"
- **THEN** the system rejects the request with a validation error naming the duplicate

#### Scenario: Names differing only by surrounding whitespace are duplicates

- **WHEN** a client creates a group with members "Ana" and " Ana "
- **THEN** the system rejects the request with a validation error naming the duplicate

### Requirement: Group retrieval

The system SHALL allow a client to retrieve a group by its identifier, returning its name, its full member list, **and its default currency**.

#### Scenario: Existing group retrieved

- **WHEN** a client requests a group by an identifier that exists
- **THEN** the system returns the group's name, its members with their identifiers, and its default currency

#### Scenario: Unknown group

- **WHEN** a client requests a group by an identifier that does not exist
- **THEN** the system responds with a not-found error

### Requirement: Membership is fixed after creation

The system SHALL NOT provide any means to add or remove members after a group is created, so that every recorded expense refers to a member that still exists.

#### Scenario: No membership mutation is exposed

- **WHEN** a client attempts to add or remove a member of an existing group
- **THEN** the system responds with a method-not-allowed or not-found error
- **AND** the group's member list is unchanged

### Requirement: Default currency is fixed after creation

The system SHALL NOT provide any means to change a group's default currency after creation, so that the currency an existing expense was recorded under can never be retroactively reinterpreted.

#### Scenario: No default-currency mutation is exposed

- **WHEN** a client attempts to change an existing group's default currency
- **THEN** the system responds with a method-not-allowed or not-found error
- **AND** the group's default currency is unchanged

#### Scenario: Recorded expenses keep their currency

- **WHEN** any group state changes
- **THEN** the currency stored on each already-recorded expense is unchanged

