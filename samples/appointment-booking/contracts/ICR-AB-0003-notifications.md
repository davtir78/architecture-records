---
icr: "0.1"
id: ICR-AB-0003
title: Notification delivery
status: agreed
version: 1.0.1
date: 2026-10-03
pattern:
  id: int-middleware-cloud
  url: https://www.itarchitecturepatterns.net/patterns/int-middleware-cloud
interaction: async-message
provider:
  system: Email and SMS delivery provider
  owner: Messaging provider (SaaS), managed by the Booking platform team
  contact: booking-platform@example.com
consumers:
  - system: Notification worker
    owner: Booking platform team
    contact: booking-platform@example.com
interfaces:
  - type: asyncapi
    ref: notifications/asyncapi.yaml
    version: 1.0.1
  - type: other
    ref: Provider's message-send API and delivery-status webhooks, linked in the notifications repository
    version: Provider-current
data:
  classification: Personal information
  personal-information: true
decisions:
  - ADR-AB-0007
---

# Notification delivery

## Purpose

Every booking, reschedule and cancellation is confirmed to the customer by email and, where they gave a mobile number, by SMS, with a reminder the day before. This integration takes booking events from the platform's queue and hands messages to an external delivery provider. If it stops, bookings still succeed, but customers are not told about them and more of them miss appointments.

## Parties and responsibilities

| Party | System | Responsible for |
|---|---|---|
| Provider | Email and SMS delivery provider | Accepting messages, delivering them, and reporting delivery status |
| Consumer | Notification worker | Turning booking events into messages, sending each exactly once, and tracking delivery |

## Interaction

- Pattern: Integration Middleware Services (Cloud). Booking events reach the worker through a queue fed by a transactional outbox (see the design notes in the [notification requirements](../requirements/notifications.md)), so the provider's availability never affects booking.
- Direction and trigger: queue to worker to provider, on every booking event and on each reminder's scheduled time; provider to worker for delivery-status callbacks.
- Volume: typical 3 messages per booking (confirmation email, confirmation SMS, reminder); peak 60 messages per second.
- Ordering: per booking, a cancellation supersedes any unsent confirmation or reminder for that booking; across bookings, order does not matter.

## Interface specification

- asyncapi `notifications/asyncapi.yaml` v1.0.1 — the booking events the worker consumes (`BookingConfirmed`, `BookingRescheduled`, `BookingCancelled`, `ReminderDue`); authoritative copy: the notifications repository.
- other: the provider's send API and delivery-status webhooks; authoritative copy: the provider's documentation, pinned by link in the notifications repository.
- Design standard followed: CloudEvents 1.0 envelopes for booking events.

## Data

- Entities: booking event, message, delivery status.
- Classification: Personal information; driven by: customer name, email address and phone number.
- Personal information: customer first name, email address and mobile number, and the booking's service, staff member's first name and time; shared with the provider solely to deliver the message.
- Residency: events and message records stored in Australia; the provider processes messages in the region set in its contract.
- Retention: provider keeps delivery logs for 30 days under its contract; consumer keeps message records for 12 months, without message bodies.

## Security

- Authentication: API key from the provider for sending; signed webhooks from the provider for delivery status.
- Credential issue and rotation: the key is held in the secrets manager and rotated every 90 days, or at once if exposed.
- Authorisation: the key is restricted to sending from the platform's verified sender domain and SMS sender id.
- Transport: TLS 1.2 or later both ways.
- Network path: outbound through the platform's egress to the provider; webhooks inbound through the API gateway with signature validation.
- Pattern controls inherited: [Integration Middleware Services (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-middleware-cloud); additional controls: SPF, DKIM and DMARC on the sender domain.

## Service levels

- Availability: the provider's contracted 99.9% monthly; the consumer queues through provider outages of up to 24 hours.
- Latency: confirmation handed to the provider within 30 seconds of the booking at p95, measured at the worker.
- Throughput: within the provider's contracted rate; beyond it, the worker slows its sending and the queue absorbs the difference.
- Freshness: a reminder is sent between 24 and 23 hours before the appointment.
- Support hours for these objectives: 24×7.

## Error handling

- Errors and their meaning: throttled means slow down; invalid recipient means the address or number is unusable and the business is shown it; provider error means retry.
- Timeouts: 10 seconds per send.
- Retries: the worker retries with exponential backoff for up to 24 hours **from when the message became due**, not from when it was queued (a reminder queued days ahead must not be given up on before its time); after that the message goes to the dead-letter queue.
- Idempotency: each message is keyed by event id and channel; the worker records every send and never sends the same key twice.
- Failed messages or files: messages still failing 24 hours after they were due go to the dead-letter queue, shown on the support dashboard, and the business sees "not delivered" against the booking.
- Recovery after an outage: the queue drains in order once the provider returns; reminders whose window has passed are dropped rather than sent late.

## Change and versioning

- Interface versioning: event types are versioned in the CloudEvents `type` (for example `booking.confirmed.v1`); new fields are additive.
- Breaking change means: removing or renaming an event field, or changing an event type's meaning.
- Notice before a breaking change or deprecation: 30 days, via the platform's internal change log (both sides are internal teams).
- Changing this ICR: the Booking platform team lead.

## Observability

- Correlation identifier: the booking event's CloudEvents `id`, carried into the provider request as a tag.
- Logs: every send and status callback, with event id, channel and outcome; never message bodies or contact details.
- Metrics and alerts: oldest unsent message older than 5 minutes, or more than 2% of sends failing over 15 minutes, alerts the Booking platform on-call.

## Support and escalation

| Party | Contact | Hours | Escalation |
|---|---|---|---|
| Booking platform team | booking-platform@example.com | 24×7 on-call | Head of engineering |
| Messaging provider | Provider support portal | 24×7 per contract | Provider account manager |

Severity levels and response targets: Sev 1 (no messages sending) 15 minutes; Sev 2 (one channel failing) 1 hour; Sev 3 (delays under 1 hour) next business day.

## Acceptance criteria

1. With the provider unavailable, bookings succeed; when it returns, every pending message is sent exactly once.
2. A booking cancelled before its confirmation is sent produces a cancellation message and no confirmation.
3. A reminder whose window passed during an outage is not sent late.
4. Replaying the same booking event sends nothing new.
5. No log line contains a message body, email address or phone number.

## Related records

- Pattern: [Integration Middleware Services (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-middleware-cloud)
- Decisions: ADR-AB-0007 (data store: the outbox is a table beside the booking)
- Requirements: [notifications](../requirements/notifications.md), with the design notes on the outbox
