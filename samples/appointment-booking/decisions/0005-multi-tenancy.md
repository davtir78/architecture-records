---
template: architecture-decision-record
status: accepted
date: 2026-09-26
decision-makers: Solution architect; Data architect
consulted: Security architect; Platform engineer
informed: Product owner; Support team
---

# ADR-AB-0005 Share one database across businesses, with row-level isolation

## Context and Problem Statement

Many businesses use the service, from a sole practitioner to a practice with thirty staff. Each business's customers, staff and bookings must be invisible to every other business. Most businesses are small, so the cost per business matters, and onboarding a new one should take minutes, not an infrastructure change.

How are businesses kept apart in storage?

## Decision Drivers

* **Isolation is non-negotiable:** a defect in one query must not leak another business's customers.
* **Onboarding is self-service** and immediate.
* **Cost per business** stays low for the many small ones.
* **One schema to migrate**, so a change ships to everyone at once.

## Considered Options

* One shared database, a business id on every row, and row-level security policies in the database
* One schema per business in a shared database
* One database per business
* A business id on every row, filtered only in application code — not taken forward, because a single missed filter leaks data, and nothing below the application would catch it.

## Decision Outcome

Chosen option: "One shared database with row-level security", because it keeps onboarding instant and cost low, and it enforces isolation in the database rather than trusting every query to remember a filter. Each request sets the business it acts for once, at the start of its database session, and the policies scope every read and write to that business.

### Consequences

* Good, because isolation is enforced by the database for every query, including ones written in a hurry.
* Good, because onboarding a business is inserting a row.
* Good, because one migration updates everyone.
* Bad, because all businesses share capacity; one very busy business can affect others, so per-business rate limits are needed at the API.
* Bad, because a business that later demands its own database needs a migration path; the design keeps the business id in every key to make that extraction possible.
* Neutral, because administrative and reporting jobs that legitimately span businesses must use a separate, audited role.

### Confirmation

| Action | Owner | Target date | Status |
| :--- | :--- | :--- | :--- |
| Automated test on every build: a session for business A reads zero rows of business B, for every table | Data architect | 2026-10-17 | not started |
| Quarterly review of the cross-business reporting role's access log | Security architect | 2026-12-31 | not started |

## Pros and Cons of the Options

### Shared database with row-level security

* Good, because isolation is centralised and testable.
* Bad, because capacity is shared.

### One schema per business

* Good, because isolation is structural.
* Bad, because every migration runs once per business, and thousands of schemas strain tooling.

### One database per business

* Good, because isolation and capacity are both complete.
* Bad, because the cost and operational load per business are far above what a small practice pays.

### Evaluation Summary

| Criterion | Shared + row-level security | Schema per business | Database per business |
| :--- | :--- | :--- | :--- |
| Security risk | low | low | low |
| Operational cost | low | high | high |
| Implementation cost | medium | high | high |
| Alignment with business outcomes | high | medium | low |

## More Information

### Governance

| | |
| :--- | :--- |
| Decision tier | A — determines how every business's data is protected |
| Initiative and phase | Appointment Booking — design |
| Endorsed | 2026-09-26, architecture review |
| Deviates from principles | None |

### Related Decisions and Documents

| Reference | Relation | Description |
| :--- | :--- | :--- |
| [Solution Architecture Design](../sad.md) | reference | Section 6, data architecture |
| [ICR-AB-0001 Booking API](../contracts/ICR-AB-0001-booking-api.md) | relates to | How a request identifies its business |
| [ADR-AB-0002 API exposure](0002-api-exposure.md) | relates to | The widget key resolves to one business |
| [ADR-AB-0007 Data store](0007-data-store.md) | depends on | Row-level security is the isolation mechanism |
