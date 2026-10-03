# Availability requirements

## Purpose

What a customer is shown as free, how fresh that must be, and how times are kept right across time zones and daylight saving. Requirements, not decisions: see [`../decisions/`](../decisions/) for how the design achieves them. The numbers in brackets are the functional requirements of the [SAD](../sad.md#21-key-functional-requirements) each one specifies.

## Requirements

### Requirement: Search shows only times that are really free (FR-1, FR-4)

The system SHALL list only times inside the staff member's working hours that do not overlap their calendar's busy time, an active hold or a booking, and that are at least one hour away.

#### Scenario: Working hours

- **WHEN** a customer searches for a service
- **THEN** only times inside the working hours of staff who provide it are listed, in steps that fit the service's length

#### Scenario: A calendar appointment

- **WHEN** a staff member adds an appointment in their own calendar
- **THEN** the overlapping times disappear from search within 60 seconds, and reappear if it is removed

#### Scenario: Short notice

- **WHEN** a time is less than an hour away
- **THEN** it is not offered

### Requirement: A confirmation is always checked against the live calendar (FR-4)

The system SHALL check the staff member's own calendar for the one time being confirmed, so that a change the synchronised copy has not yet seen is still caught.

#### Scenario: A change the copy has not seen

- **WHEN** an event is added to a staff calendar and the change has not reached search
- **THEN** search still shows the time free, and confirming it is refused as no longer free, with the next free times offered at once, and no messages are queued

#### Scenario: The calendar provider is unavailable

- **WHEN** the live check cannot be made because the provider is down
- **THEN** the booking is confirmed anyway, the skipped check is recorded, and the calendar is reconciled when the provider returns

This last scenario accepts a risk deliberately: a double booking against an event the system has not seen. It follows the principle that a provider outage degrades the experience and never stops a booking.

**Settled.** [SAD section 8.2](../sad.md#82-disaster-recovery-dr--business-continuity), [ADR-AB-0003](../decisions/0003-availability-source-of-truth.md) and the contracts all say the same: confirm, and reconcile afterwards. The alternative, refusing the booking, is recorded in the decision as the rejected option. The product owner's acceptance of the risk is an action in that decision.

### Requirement: A booking is one real moment, and working hours follow the wall clock (FR-1)

The system SHALL record each booking as one unambiguous moment, SHALL interpret working hours in the business's own time zone, and SHALL stay correct when clocks change.

#### Scenario: Nine to five all year

- **WHEN** daylight saving starts or ends
- **THEN** a 9 am start is still 9 am at the business, though its moment has shifted by an hour

#### Scenario: The skipped hour

- **WHEN** clocks go forward and an hour does not exist
- **THEN** no time is offered inside it

#### Scenario: The repeated hour

- **WHEN** clocks go back and an hour happens twice
- **THEN** each moment in it is offered once

#### Scenario: A customer elsewhere

- **WHEN** a customer in another time zone views the business's times
- **THEN** each time is shown in the customer's zone, and the zone is named beside it

### Requirement: Every time shown names its zone (FR-1)

The system SHALL show a time of day only with the time zone it is in.

#### Scenario: Reading a time

- **WHEN** a time is shown to a customer
- **THEN** it reads like "Tuesday 10:00 am AEDT", never a bare clock time

## Design notes

How the design keeps times right. Not a decision record: the alternatives are weak and the choice is conventional.

- **Chosen:** store each booking as a UTC instant; store working hours as rules in the business's IANA time zone; expand the rules into instants for each date in that zone, so the time zone database handles the changeover; convert for display.
- **Rejected: everything in the business's local time.** A booking would be ambiguous in the repeated hour and could not be compared across zones.
- **Rejected: everything in UTC, including working hours.** "9 to 5 UTC" drifts by an hour against the wall clock whenever daylight saving changes.
- **Cost accepted:** every display needs a conversion, every test needs dates on both sides of a changeover, and the time zone database must be kept up to date as governments change their rules.
