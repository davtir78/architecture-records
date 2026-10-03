# Notification requirements

## Purpose

What customers are told about their booking, and what must hold when the service that delivers messages is not working. Requirements, not decisions: see the design notes at the end for how the design delivers them. The numbers in brackets are the functional requirements of the [SAD](../sad.md#21-key-functional-requirements) each one specifies.

## Requirements

### Requirement: The customer is told about every booking and every change (FR-5)

The system SHALL send a confirmation when a booking is made, and a message when it is rescheduled or cancelled, by email, and also by SMS when the customer gave a mobile number.

#### Scenario: A new booking

- **WHEN** a booking is confirmed
- **THEN** the customer receives a confirmation by email, and by SMS if they gave a number

#### Scenario: No mobile number

- **WHEN** a customer did not give a phone number
- **THEN** they receive email only

#### Scenario: A change

- **WHEN** a booking is rescheduled or cancelled
- **THEN** the customer receives a message saying so

### Requirement: A reminder arrives the day before, and never late (FR-5)

The system SHALL send a reminder 24 hours before the appointment, SHALL NOT schedule one for a booking made inside that day, and SHALL drop a reminder whose useful window has passed instead of sending it late.

#### Scenario: A booking days ahead

- **WHEN** an appointment is more than a day away
- **THEN** a reminder is sent a day before it

#### Scenario: A booking made on the day

- **WHEN** an appointment is booked less than 24 hours ahead
- **THEN** no reminder is scheduled

#### Scenario: A reminder that would arrive late

- **WHEN** a reminder is still unsent less than two hours before the appointment
- **THEN** it is dropped

### Requirement: A messaging outage never stops a booking

The system SHALL confirm bookings whether or not messages can be delivered, SHALL keep every message until it has been delivered, and SHALL send each message exactly once when the provider returns.

#### Scenario: The provider is down

- **WHEN** the delivery provider is unavailable while bookings are made
- **THEN** every booking succeeds, and the messages wait

#### Scenario: The provider returns

- **WHEN** the provider comes back
- **THEN** each waiting message is sent exactly once

#### Scenario: A replay

- **WHEN** the same booking event is processed again
- **THEN** no second message is sent

### Requirement: A cancellation replaces what has not been sent

The system SHALL, when a booking is cancelled or moved, withdraw any confirmation or reminder about it that has not yet been sent.

#### Scenario: Cancelled before the confirmation went out

- **WHEN** a booking is cancelled while its confirmation is still waiting
- **THEN** the customer receives a cancellation and no confirmation

#### Scenario: Moved before the confirmation went out

- **WHEN** a booking is moved while its confirmation is still waiting
- **THEN** the customer receives the "moved" message and no confirmation of the old time

### Requirement: Failures are visible, and not endless

The system SHALL stop retrying a message after 24 hours of trying, SHALL show the business that it was not delivered, and SHALL never put a customer's contact details or a message's text in a log.

#### Scenario: A message that cannot be delivered

- **WHEN** a message is still failing 24 hours after it became due, or its address is refused
- **THEN** it is set aside, and the booking shows as not delivered

#### Scenario: Reading the logs

- **WHEN** a booking, a change and a cancellation are made with distinctive contact details
- **THEN** none of those details, and no message text, appears in any log line

## Design notes

How the design delivers these. Not a decision record: it is a pattern applied inside one subsystem, cheap to change, and the run cost is small.

- **Chosen: a transactional outbox and a queue.** The booking and a row describing the event to publish are written in the same database transaction. A relay moves those rows to a queue, and a notification worker calls the delivery provider with retries. A booking therefore cannot exist without its message, or a message without its booking, and the provider's state cannot affect the booking. The same events also feed calendar synchronisation, with no second mechanism. Library patterns: [Integration Middleware Services (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-middleware-cloud) for the calls to the provider.
- **Rejected: call the provider directly in the booking request.** The simplest thing that works on a good day, but a slow provider makes booking slow, and a down provider forces a choice between failing the booking and losing the message.
- **Rejected: publish to the queue straight after committing the booking.** A crash between the commit and the publish loses the message silently.
- **Cost accepted:** delivery is at-least-once, so the worker sends idempotently (each message carries the event id, and the worker records what it has sent); the relay is one more component to run and watch; confirmations arrive seconds after booking, which the widget's own confirmation screen covers.
- **The outbox is a table in the booking database**, so it depends on the choice of data store ([ADR-AB-0007](../decisions/0007-data-store.md)).
