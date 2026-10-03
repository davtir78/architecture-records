---
icr: "0.1"
id: ICR-AB-0002
title: Calendar synchronisation
status: proposed
version: 0.10.0
date: 2026-10-03
pattern:
  id: int-native-cloud
  url: https://www.itarchitecturepatterns.net/patterns/int-native-cloud
interaction: async-event
provider:
  system: Staff calendar providers (Google Calendar, Microsoft 365)
  owner: Each business's calendar administrator
  contact: Set per business in the admin console
consumers:
  - system: Calendar sync service
    owner: Booking platform team
    contact: booking-platform@example.com
interfaces:
  - type: other
    ref: Provider calendar APIs and change-notification webhooks, as linked in the calendar-sync repository
    version: Provider-current
data:
  classification: Internal
  personal-information: true
decisions:
  - ADR-AB-0003
  - ADR-AB-0004
---

# Calendar synchronisation

## Purpose

Staff keep their own calendars in Google Calendar or Microsoft 365. This integration keeps a copy of each staff member's busy times so the widget can show real availability, and writes each booking into the staff member's calendar so they see who is coming. If it stops, availability goes stale and bookings stop appearing in staff calendars; every confirmation is still checked live while the provider can be reached, so a stale copy does not cause a double booking. The exception is a provider outage at the moment of confirmation: the booking is then confirmed without the check and reconciled afterwards (see Error handling).

## Parties and responsibilities

| Party | System | Responsible for |
|---|---|---|
| Provider | Staff calendar providers | Change notifications, the calendar APIs, and each staff member's consent to connect |
| Consumer | Calendar sync service | Subscribing, renewing subscriptions, keeping busy times current, writing bookings to calendars |

## Interaction

- Pattern: Integration Native Connectors (Cloud). Each provider's own API and change notifications are used directly, rather than through a generic middleware layer, because both providers already offer a supported, well-documented change feed. The reasoning is recorded in ADR-AB-0004.
- Direction and trigger: provider to consumer when a calendar changes (a notification, followed by a delta query); consumer to provider when a booking is confirmed, rescheduled or cancelled.
- Volume: typical 2 notifications per staff member per hour; peak 30 per staff member per hour (Monday mornings).
- Ordering: notifications are treated as "something changed", never as the change itself; the delta query returns the current state, so out-of-order notifications do no harm.

## Interface specification

- other: each provider's calendar API and change-notification webhook — busy times, events written by the platform; authoritative copy: the provider's published documentation, pinned by link in the calendar-sync repository.
- Design standard followed: delta queries driven by notifications, with a full re-synchronisation every 24 hours as the safety net.

## Data

- Entities: busy interval (start, end, busy/free), platform-written booking event.
- Classification: Internal; driven by: staff availability patterns.
- Personal information: the booking events written to a staff calendar hold the customer's first name and the service booked, so the staff member knows who is coming; busy intervals read from calendars hold no titles, attendees or descriptions.
- Residency: busy intervals stored in Australia; booking events live in each provider's region for that business.
- Retention: provider per its own policy; consumer keeps busy intervals for 90 days ahead and deletes past intervals nightly.

## Security

- Authentication: OAuth 2.0 authorisation-code flow, granted by each staff member when they connect their calendar.
- Credential issue and rotation: refresh tokens are stored encrypted in the secrets manager and refreshed by the provider's rules; revoking access in the admin console deletes them immediately.
- Authorisation: the narrowest scopes each provider offers for reading free/busy and writing events to one calendar; no mailbox or contacts access.
- Transport: TLS 1.2 or later both ways; notification endpoints validate each provider's signature or client-state token.
- Network path: public internet to and from the providers; the notification endpoint is behind the API gateway.
- Pattern controls inherited: [Integration Native Connectors (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-native-cloud); additional controls: minimum-scope review before any scope change.

## Service levels

- Availability: best effort on the providers' side; the consumer tolerates a provider outage of up to 24 hours without data loss.
- Latency: a calendar change is reflected in availability within 60 seconds at p95, measured from the notification's arrival.
- Throughput: within each provider's published per-user and per-application limits; beyond them the consumer backs off as the provider instructs.
- Freshness: 60 seconds p95 for changes; 24 hours worst case, bounded by the full re-synchronisation.
- Support hours for these objectives: 24×7 for the consumer; the providers' own support terms apply to them.

## Error handling

- Errors and their meaning: throttling responses mean back off; an expired or revoked grant means the staff member must reconnect; a missing subscription means it lapsed and is recreated.
- Timeouts: 10 seconds per provider call.
- Retries: the consumer retries with exponential backoff and jitter, up to 1 hour, honouring any retry-after hint; after that the change waits for the next full re-synchronisation.
- Idempotency: events written to calendars carry the booking id as an extended property; writing the same booking twice updates, never duplicates.
- Failed messages or files: failures after retries go to the sync dead-letter queue, and the business administrator is emailed when a staff member's calendar has not synchronised for 1 hour.
- Recovery after an outage: the consumer asks a failing provider once a minute, however long the retry backoff has grown; when it answers, a full re-synchronisation for the affected staff members runs and pending writes are retried at once, so nothing waits out the backoff.
- The live check at confirmation: the consumer asks the provider about the one time being confirmed, with the same ten-second timeout. If the provider cannot answer, the answer is "not checked", not "free"; the booking proceeds, the skipped check is logged, and the booking's calendar event is written, and any conflict found, when the provider returns. This accepts a possible double booking; see ADR-AB-0003.
- Subscription renewal: a change subscription is renewed when less than a day of its life remains (the demo uses a three-day subscription). A subscription that lapses anyway raises an alert, is created again, and triggers a full re-synchronisation, because changes may have been missed.

## Change and versioning

- Interface versioning: provider API versions are pinned in configuration and reviewed quarterly.
- Breaking change means: a provider retiring an API version, a scope, or its change-notification mechanism.
- Notice before a breaking change or deprecation: as published by each provider (typically 12 months); the platform tracks their deprecation notices.
- Changing this ICR: the Booking platform team lead, with the security architect for any scope change.

## Observability

- Correlation identifier: the staff member's connection id, plus the booking id for events written.
- Logs: every notification, delta query and write, with connection id and outcome; never event titles or attendee details.
- Metrics and alerts: age of the oldest unsynchronised change per business above 5 minutes alerts the Booking platform on-call; a connection failing for 1 hour emails the business administrator.

## Support and escalation

| Party | Contact | Hours | Escalation |
|---|---|---|---|
| Booking platform team | booking-platform@example.com | 24×7 on-call | Head of engineering |
| Business calendar administrator | Set per business | Business's own hours | Business owner |

Severity levels and response targets: Sev 1 (sync stopped for all businesses) 15 minutes; Sev 2 (one provider or one business) 1 hour; Sev 3 (one staff member) next business day.

## Acceptance criteria

1. A new appointment added directly to a staff member's calendar removes the overlapping slots from search within 60 seconds.
2. Confirming a booking creates exactly one event in the staff member's calendar, and confirming the same booking again creates no second event.
3. Revoking a staff member's calendar access in the admin console deletes their stored tokens and stops synchronisation within 1 minute.
4. With a provider unavailable for 1 hour, no change is lost: every calendar is consistent within 10 minutes of the provider returning.
5. No stored busy interval contains an event title, attendee or description.
6. A subscription with less than a day left is renewed; one that lapsed raises an alert, is recreated, and the calendar is re-synchronised.
7. With the provider unavailable at confirmation the booking succeeds and the check is recorded as skipped; with it available the live check refuses a conflicting time.

## Related records

- Pattern: [Integration Native Connectors (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-native-cloud)
- Decisions: ADR-AB-0003 (availability source of truth), ADR-AB-0004 (build the connectors rather than buy)
- Requirements: [availability](../requirements/availability.md), [notifications](../requirements/notifications.md) (the booking events it consumes)

## Open issues

- Microsoft 365 change-notification subscriptions for calendar events expire after a few days and must be renewed well before they lapse. A renewal schedule and an alert on a missed renewal are now designed (see Error handling) and shown working against a stand-in provider. This contract stays "proposed" until the platform lead agrees them and they are tried against a real provider, whose actual subscription lifetime and renewal limits may differ.
