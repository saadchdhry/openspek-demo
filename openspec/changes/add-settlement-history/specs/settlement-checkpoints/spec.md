## Purpose

Records the moments a group declared itself settled, dividing its expense history into periods so that balances reflect what is outstanding now rather than everything ever recorded.

## ADDED Requirements

### Requirement: Recording a checkpoint

The system SHALL allow a client to record a settlement checkpoint against a group, optionally with a free-text note. The system SHALL stamp each checkpoint with the time it was recorded and SHALL treat that time as the boundary of the settled period.

#### Scenario: Checkpoint recorded

- **WHEN** a client records a checkpoint against a group
- **THEN** the system stores it with the current time and returns its identifier

#### Scenario: Note is optional

- **WHEN** a client records a checkpoint without a note
- **THEN** the system stores the checkpoint with an empty note

### Requirement: Checkpoints are append-only

The system SHALL NOT provide any means to edit or delete a recorded checkpoint. A group's settled history SHALL only ever grow.

#### Scenario: Edit is refused

- **WHEN** a client attempts to modify an existing checkpoint
- **THEN** the system responds with a method-not-allowed error

#### Scenario: Delete is refused

- **WHEN** a client attempts to delete an existing checkpoint
- **THEN** the system responds with a method-not-allowed error

### Requirement: Checkpoint history

The system SHALL allow a client to list a group's checkpoints in reverse chronological order, each with its time and note.

#### Scenario: History listed

- **WHEN** a client lists the checkpoints of a group with three recorded
- **THEN** the system returns all three, newest first

#### Scenario: Group never settled

- **WHEN** a client lists the checkpoints of a group with none
- **THEN** the system returns an empty list rather than an error

### Requirement: A checkpoint asserts nothing about payment

The system SHALL treat a checkpoint as a claim recorded on behalf of the group, not as evidence that any transfer occurred. The system SHALL NOT represent a checkpoint as a confirmed or verified payment in any response.

#### Scenario: Checkpoint is not evidence

- **WHEN** a client reads a checkpoint
- **THEN** the response describes it as group-asserted
- **AND** the response contains no claim that money moved
