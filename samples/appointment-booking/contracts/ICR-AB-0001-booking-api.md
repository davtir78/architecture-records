---
icr: "0.1"
id: ICR-AB-0001
title: Booking API
status: agreed
version: 1.1.0
date: 2026-10-03
pattern:
  id: int-api-external
  url: https://www.itarchitecturepatterns.net/patterns/int-api-external
interaction: sync-request-response
provider:
  system: Booking API
  owner: Booking platform team
  contact: booking-platform@example.com
consumers:
  - system: Booking widget
    owner: Booking front-end team
    contact: booking-frontend@example.com
interfaces:
  - type: openapi
    ref: booking-api/openapi.yaml
    version: 1.1.0
data:
  classification: Personal information
  personal-information: true
decisions:
  - ADR-AB-0001
  - ADR-AB-0002
  - ADR-AB-0003
  - ADR-AB-0005
---

# Booking API

## Purpose

The booking widget, running inside a business's own website, uses this API to show a business's services and free times and to hold, confirm, reschedule and cancel bookings. It is the only way customers book; if it stops, no business on the platform can take a booking online.

## Parties and responsibilities

| Party | System | Responsible for |
|---|---|---|
| Provider | Booking API | Availability, holds and bookings; enforcing business isolation and rate limits; this contract's service levels |
| Consumer | Booking widget | Calling only the operations below; showing every time with its zone; offering alternatives when a slot is taken |

## Interaction

- Pattern: Integration API Management (External). The widget calls across the public internet from a customer's browser, through the API gateway. Why the interface takes this form is recorded in ADR-AB-0002.
- Direction and trigger: widget to API, on each customer action.
- Volume: typical 20 requests per second across all businesses; peak 200 per second (Monday mornings).
- Ordering: not applicable; each request stands alone, and concurrency on a slot is resolved by the provider (see the [booking requirements](../requirements/booking.md)).

## Interface specification

- openapi `booking-api/openapi.yaml` v1.1.0 — services, availability search, holds, bookings; authoritative copy: the booking-api repository, published to the API developer portal.
- Design standard followed: resource-oriented REST, JSON, ISO 8601 instants with offsets (see the [availability requirements](../requirements/availability.md)), RFC 9457 problem details for errors.

Operations marked *proposed* were found necessary while building the demo and are not yet agreed by the front-end and platform leads:

| Operation | Purpose |
|---|---|
| `GET /businesses/{businessId}/services` | Services a business offers, with durations |
| `GET /businesses/{businessId}/availability?service=&from=&to=` | Free slots, per staff member, as UTC instants |
| `POST /businesses/{businessId}/holds` | Hold a slot for five minutes |
| `POST /holds/{holdId}/extend` | Add five minutes to a hold, up to ten times (proposed) |
| `DELETE /holds/{holdId}` | Release a hold the customer no longer wants (proposed) |
| `POST /holds/{holdId}/confirm` | Confirm a hold as a booking, with the customer's details |
| `PATCH /bookings/{bookingId}` | Reschedule, using the customer's manage-booking token |
| `DELETE /bookings/{bookingId}` | Cancel, using the customer's manage-booking token |

## Data

- Entities: service, slot, hold, booking, customer contact.
- Classification: Personal information; driven by: customer name, email address and phone number.
- Personal information: customer name, email and mobile number, collected to confirm and remind the customer of their booking and shared only with the business they booked with.
- Residency: stored in Australia.
- Retention: provider keeps bookings and contact details for 24 months after the appointment, then deletes them; the consumer keeps nothing (the widget holds no data beyond the page).

## Security

- Authentication: each request carries the business's public widget key, which identifies the business and is restricted to the domains the business registers; manage-booking operations also require the customer's per-booking token, sent in their confirmation message.
- Credential issue and rotation: widget keys are issued in the admin console and rotated by the business at any time; booking tokens are single-purpose and expire when the appointment has passed.
- Authorisation: the gateway resolves the key to one business, and the service scopes every query to it through row-level security (ADR-AB-0005).
- Transport: TLS 1.2 or later; HSTS on the API domain.
- Network path: public internet to the API gateway; nothing behind the gateway is exposed.
- Pattern controls inherited: [Integration API Management (External)](https://www.itarchitecturepatterns.net/patterns/int-api-external); additional controls: registered-origin check on the widget key, bot protection on hold and confirm.

## Service levels

- Availability: 99.9% per month, measured at the API gateway.
- Latency: availability search p95 ≤ 400 ms; confirm p95 ≤ 800 ms (it includes the live calendar check), measured at the gateway.
- Throughput: 50 requests per second per business; beyond it, `429 Too Many Requests` with `Retry-After`.
- Freshness: availability reflects staff calendar changes within 60 seconds, and a confirmation is checked live whenever the calendar provider can be reached (ADR-AB-0003). When it cannot, the booking is confirmed without the check, the skipped check is logged, and the calendar is reconciled when the provider returns; the possible double booking is an accepted risk, recorded in ADR-AB-0003.
- Support hours for these objectives: 24×7.

## Error handling

- Errors and their meaning: `400` invalid request; `401` unknown or revoked key; `403` origin not registered; `404` unknown resource; `409` slot no longer free, or a hold already confirmed, or extended too often (the response lists the next free slots); `410` hold expired; `422` an idempotency key reused for a different request; `429` rate limited; `503` temporarily unavailable.
- Timeouts: the widget waits 10 seconds, then shows a retry prompt.
- Retries: the widget retries only safe requests (`GET`) and requests carrying an idempotency key, at most twice, with jittered backoff; it never retries a `409`.
- Idempotency: `POST` requests carry an `Idempotency-Key` header; the provider returns the original result for a repeated key within 24 hours, and answers `422` if the key is reused for a different request. Confirming a hold that is already confirmed, with a new key, is a `409` listing alternatives, not a second booking.
- Failed messages or files: not applicable; the interaction is synchronous.
- Recovery after an outage: holds that expired during the outage are released; the widget re-runs the customer's last search.
- Holds: a hold lasts five minutes, and the customer can extend it up to ten times (WCAG 2.2.1 requires a way to extend a time limit) or release it by going back, so they do not block their own first choice.

## Change and versioning

- Interface versioning: the major version is in the path (`/v1/`); additive changes ship without notice.
- Breaking change means: removing or renaming a field or operation, changing a type or an error code's meaning, or tightening validation.
- Notice before a breaking change or deprecation: 90 days, via the API developer portal and the widget release notes.
- Changing this ICR: the Booking platform and Booking front-end team leads, together.

## Observability

- Correlation identifier: `traceparent` (W3C Trace Context), created by the widget and returned in every response.
- Logs: gateway access logs and service logs carry the correlation identifier and business id, never customer contact details.
- Metrics and alerts: 5xx rate above 1% for 5 minutes, and p95 latency above target for 10 minutes, page the Booking platform on-call.

## Support and escalation

| Party | Contact | Hours | Escalation |
|---|---|---|---|
| Booking platform team | booking-platform@example.com | 24×7 on-call | Head of engineering |
| Booking front-end team | booking-frontend@example.com | Business hours, Sydney | Booking platform on-call |

Severity levels and response targets: Sev 1 (no bookings possible) 15 minutes; Sev 2 (one business or one operation failing) 1 hour; Sev 3 (degraded) next business day.

## Acceptance criteria

1. A search for one service over seven days returns in under 400 ms at p95 under the typical load.
2. Fifty simultaneous confirmations of one hold's slot produce exactly one booking and forty-nine `409` responses listing alternatives.
3. A request from a domain not registered for the widget key receives `403`.
4. A repeated `POST` with the same idempotency key creates nothing new and returns the original result.
5. Every time in every response is an ISO 8601 instant with an offset.
6. No log line contains a customer's name, email address or phone number.
7. With the calendar provider unavailable, a confirmation succeeds, the response is the same as when the check passes, and the skipped check appears in the log.
8. Extending a hold adds five minutes, an eleventh extension is refused, and releasing a hold frees its time at once.

## Related records

- Pattern: [Integration API Management (External)](https://www.itarchitecturepatterns.net/patterns/int-api-external)
- Decisions: ADR-AB-0001 (embed mechanism), ADR-AB-0002 (API exposure), ADR-AB-0003 (availability source of truth), ADR-AB-0005 (tenancy), ADR-AB-0007 (data store)
- Requirements: [booking](../requirements/booking.md), [availability](../requirements/availability.md)
