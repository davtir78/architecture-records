---
template: architecture-decision-record
status: accepted
date: 2026-10-02
decision-makers: Solution architect; Data architect
consulted: Lead back-end engineer; Security architect; Platform engineer
informed: Product owner; Support team
---

# ADR-AB-0007 Hold the booking data in a managed PostgreSQL database

## Context and Problem Statement

The platform keeps businesses, services, staff, working hours, holds, bookings, synchronised busy times and an outbox of messages. Several requirements are guarantees about this data, not about screens: no two bookings overlap, one business can never read another's data, and a booking and its messages are written together or not at all. The data must stay in Australia.

Which database holds it, given that those guarantees are far safer when the database enforces them than when application code is trusted to?

## Decision Drivers

* **The database must be able to refuse an overlap** between two time ranges for one staff member ([booking requirements](../requirements/booking.md)).
* **Isolation between businesses** that survives a bug in application code ([ADR-AB-0005](0005-multi-tenancy.md)).
* **A booking and its messages in one transaction** (the outbox described in the [notification requirements](../requirements/notifications.md)).
* **Managed:** backups, patching and failover are not something a small team should run.
* **Residency:** an Australian region.
* **Scale is modest:** 200 requests a second at peak across all businesses.

## Considered Options

* A managed PostgreSQL database
* A managed document or key-value store, using conditional writes
* A managed MySQL or MariaDB database
* A distributed SQL database — not taken forward, because its cost and complexity are far beyond a peak of 200 requests a second

## Decision Outcome

Chosen option: "A managed PostgreSQL database", because it is the only candidate that can express all three guarantees inside the database: an exclusion constraint over a staff member and a time range, row-level security per business, and one transaction across a booking and its outbox rows.

### Consequences

* Good, because the no-overlap rule becomes an exclusion constraint that holds even if application servers race, and cannot be bypassed by a defect.
* Good, because row-level security makes isolation a property of the database, which is tested on every build.
* Good, because the transactional outbox needs nothing but a table.
* Bad, because writes go to one primary. That is far above the need at 200 requests a second, and search can use read replicas, but it is a ceiling to remember.
* Bad, because the design depends on PostgreSQL features. A different database would mean redesigning the overlap rule and the isolation, not swapping a driver.
* Neutral, because the major clouds all offer managed PostgreSQL in an Australian region.

### Confirmation

| Action | Owner | Target date | Status |
| :--- | :--- | :--- | :--- |
| A test writes an overlapping row directly to the database and expects it refused | Lead back-end engineer | 2026-10-17 | not started |
| A test reads one business's rows with another business's identity and expects none | Security architect | 2026-10-17 | not started |
| Load test at 200 requests a second with search on a read replica | Platform engineer | 2026-10-31 | not started |

## Pros and Cons of the Options

### Managed PostgreSQL

* Good, because the three guarantees are enforced by the database.
* Good, because it is widely understood and widely available as a managed service.
* Bad, because of the single-primary write ceiling.

### Managed document or key-value store

* Good, because it scales writes almost without limit.
* Bad, because a range-overlap rule has to be built from conditional writes on pre-split time slots, which is awkward and easy to get wrong.
* Bad, because isolation and multi-record transactions fall back on application code.

### Managed MySQL or MariaDB

* Good, because it is relational and widely available.
* Bad, because it has no exclusion constraint or row-level security, so the first two guarantees would rest on application code again.

### Evaluation Summary

| Criterion | PostgreSQL | Document store | MySQL |
| :--- | :--- | :--- | :--- |
| Database-enforced no-overlap | yes | no | no |
| Database-enforced tenant isolation | yes | no | no |
| Booking and messages in one transaction | yes | limited | yes |
| Write scale | medium | high | medium |

## More Information

The demo of this system runs on SQLite, with a trigger standing in for the exclusion constraint and application code standing in for row-level security. Its README states what that does and does not prove: the overlap rule is genuinely enforced by the database, but isolation is by code, which is weaker than this decision calls for.

### Governance

| | |
| :--- | :--- |
| Decision tier | A — the data store is costly to change and underpins three requirements |
| Initiative and phase | Appointment Booking — design |
| Endorsed | 2026-10-02, architecture review |
| Deviates from principles | None |

### Related Decisions and Documents

| Reference | Relation | Description |
| :--- | :--- | :--- |
| [ADR-AB-0005 Multi-tenancy](0005-multi-tenancy.md) | enables | Row-level security is the isolation mechanism |
| [Notification requirements](../requirements/notifications.md) | enables | The outbox, in the design notes, is a table in this database |
| [ADR-AB-0006 Hosting and recovery](0006-hosting-and-recovery.md) | relates to | The database is managed, multi-zone, with point-in-time recovery |
| [Booking requirements](../requirements/booking.md) | reference | The no-overlap guarantee |
