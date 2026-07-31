# group-management Specification

## Purpose
Defines the group: the named container that a fixed set of members records shared expenses against. Every expense and every balance in Splitr is scoped to exactly one group.
## Requirements
### Requirement: Group creation

The system SHALL allow a client to create a group with a display name and an initial list of members. The system SHALL assign each group a unique identifier and each member a unique identifier scoped to that group.

#### Scenario: Group created with members

- **WHEN** a client creates a group named "Lisbon trip" with members "Ana", "Ben", and "Cara"
- **THEN** the system returns a group identifier and three member identifiers
- **AND** the group's member list contains exactly those three members

#### Scenario: Group name is required

- **WHEN** a client creates a group with an empty or whitespace-only name
- **THEN** the system rejects the request with a validation error
- **AND** no group is created

#### Scenario: Group requires at least two members

- **WHEN** a client creates a group with fewer than two members
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

The system SHALL allow a client to retrieve a group by its identifier, returning its name and full member list.

#### Scenario: Existing group retrieved

- **WHEN** a client requests a group by an identifier that exists
- **THEN** the system returns the group's name and its members with their identifiers

#### Scenario: Unknown group

- **WHEN** a client requests a group by an identifier that does not exist
- **THEN** the system responds with a not-found error

### Requirement: Membership is fixed after creation

The system SHALL NOT provide any means to add or remove members after a group is created, so that every recorded expense refers to a member that still exists.

#### Scenario: No membership mutation is exposed

- **WHEN** a client attempts to add or remove a member of an existing group
- **THEN** the system responds with a method-not-allowed or not-found error
- **AND** the group's member list is unchanged

