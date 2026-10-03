# Booking requirements

## Purpose

What the system must do for a customer who books, changes or cancels a time, and the guarantees it keeps while many people do so at once. These are requirements, not decisions: each says *what* must be true and can be checked by a test. The decisions in [`../decisions/`](../decisions/) say *how* the design makes them true, and cite these as their drivers.

Written in the same form as the site's OpenSpec proposals: each requirement says what the system SHALL do, and each scenario is a checkable WHEN and THEN. The numbers in brackets are the functional requirements of the [SAD](../sad.md#21-key-functional-requirements) each one specifies.

## Requirements

### Requirement: A customer books a free time without an account (FR-2)

The system SHALL let a customer choose a free time with a chosen staff member or with anyone available, enter their name, email address and, optionally, a phone number, and confirm, without creating an account.

#### Scenario: Booking a time

- **WHEN** a customer chooses a free time and gives their details
- **THEN** the booking is confirmed, and they are given a link to manage it

#### Scenario: No account

- **WHEN** a customer books
- **THEN** they are never asked to sign in or to choose a password

### Requirement: No two active bookings overlap for one staff member

The system SHALL never confirm or hold two overlapping times for the same staff member, however many requests arrive at once, and that guarantee SHALL rest on the data store, not only on application code.

#### Scenario: Two customers, one time

- **WHEN** two customers choose the same free time within the same second
- **THEN** one of them holds it, and the other is told it is taken and offered the next free times

#### Scenario: Fifty at once

- **WHEN** fifty requests ask to hold the same time simultaneously
- **THEN** exactly one succeeds, and the other forty-nine receive a conflict that lists alternative times

#### Scenario: Going round the application

- **WHEN** a row overlapping an active booking is written directly to the data store
- **THEN** the data store refuses it

#### Scenario: Different people

- **WHEN** two customers choose the same time with different staff members
- **THEN** both succeed

### Requirement: A chosen time is held while the customer completes the form (FR-2)

The system SHALL hold a chosen time for five minutes, SHALL let the customer extend the hold, and SHALL release it when it expires or when the customer lets it go.

#### Scenario: Typing slowly

- **WHEN** a customer has chosen a time and is entering their details
- **THEN** nobody else can book that time for five minutes

#### Scenario: Running out of time

- **WHEN** the five minutes pass without confirmation
- **THEN** the time is free for others, and confirming the old hold is refused as expired

#### Scenario: Extending

- **WHEN** the customer asks to keep holding the time
- **THEN** the hold lasts five more minutes from that moment, and can be extended at least ten times

#### Scenario: Changing their mind

- **WHEN** a customer goes back to choose a different time
- **THEN** the first time is released at once

### Requirement: Only the customer who booked can change or cancel (FR-3)

The system SHALL let a booking be rescheduled or cancelled only with the token sent in its confirmation message, and SHALL stop honouring the token once the appointment has passed.

#### Scenario: A wrong link

- **WHEN** someone tries to change a booking without its token, or with another booking's token
- **THEN** the request is refused

#### Scenario: Moving a booking

- **WHEN** a customer moves a booking to another free time for the same service
- **THEN** the booking is at the new time, and the old time is free again

#### Scenario: After the appointment

- **WHEN** the appointment time has passed
- **THEN** the booking can no longer be changed or cancelled

#### Scenario: Cancelling

- **WHEN** a customer cancels
- **THEN** the time is free again for other customers

### Requirement: Repeating a request is safe

The system SHALL return the original result when a request that changes anything is repeated with the same idempotency key within 24 hours, and SHALL refuse the same key used for a different request.

#### Scenario: A double click

- **WHEN** the same hold request is sent twice with the same key
- **THEN** one hold exists, and both requests receive it

#### Scenario: A reused key

- **WHEN** a key is reused for a different request
- **THEN** the second request is refused, not guessed at

## Design notes

These record how the design keeps the no-overlap guarantee. They are not a decision record: the choice is small, local to the booking service, and cheap to change.

- **Chosen:** a *hold* is created when a time is chosen, and holds and bookings are rows in one table with a data-store constraint rejecting any row that overlaps an active one for the same staff member. The constraint makes a double booking impossible whatever the application does; the hold gives the first customer the time while they type.
- **Rejected: check in application code, then insert.** Two requests can both pass the check before either inserts, which is the race this exists to prevent, and a customer can lose the time while typing.
- **Rejected: lock the staff member's row for the whole form.** Correct, but one slow typist blocks every other booking for that person, and long-held locks cause incidents.
- **Cost accepted:** an abandoned hold blocks its time for up to five minutes, and the constraint needs a data store that can express it (the choice of store is [ADR-AB-0007](../decisions/0007-data-store.md)).
