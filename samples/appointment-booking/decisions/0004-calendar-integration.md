---
template: architecture-decision-record
status: accepted
date: 2026-10-02
decision-makers: Solution architect; Lead back-end engineer; Head of engineering
consulted: Security architect; Data architect; Finance partner
informed: Product owner; Support team
---

# ADR-AB-0004 Build calendar connectors on each provider's own API, rather than buy an aggregation platform

## Context and Problem Statement

Staff keep their own calendars in Google Calendar or Microsoft 365. The platform needs to know when they are busy, so it does not offer a time they cannot take, and it must write each booking into the calendar so the staff member sees who is coming. Two ways to get there differ sharply in cost: build the connectors ourselves, against each provider's published API and change notifications, or buy access through a calendar-aggregation service that hides the providers behind one interface and charges for it.

Do we build and run the calendar connectors, or buy them, given that the cost of one grows with every staff member connected and the other is paid in engineering time up front and in upkeep?

## Decision Drivers

* **Cost that scales the right way:** an aggregation service typically charges per connected calendar, so the bill grows with every staff member of every business; our own connectors cost the same to run at ten calendars and at ten thousand.
* **Fresh within 60 seconds** ([availability requirements](../requirements/availability.md)).
* **Least privilege:** read free/busy and write events to one calendar, nothing about mail or contacts, and no third party holding those grants.
* **Only two providers are needed** for launch and for the foreseeable future.
* **Operable by a small team:** few moving parts, and failures that announce themselves.

## Considered Options

* Build a connector service on each provider's own API and change notifications, with a delta query on each notification and a full re-synchronisation every 24 hours
* Buy access through a calendar-aggregation platform
* Poll every calendar every few minutes with no notifications (a cheaper variant of building) — not taken forward, because freshness would be only as good as the polling interval, and polling every calendar every minute exhausts the providers' limits at modest scale
* Read each staff member's published calendar feed, read-only — not taken forward, because bookings could not be written back into the calendar, which the staff need

## Decision Outcome

Chosen option: "Build on each provider's own API", because the cost of buying grows with every connected calendar while the cost of building does not, both providers already offer a supported change feed and narrow scopes, and only two providers are needed, so the build is small and bounded.

A notification only says that something changed. The connector then asks the provider for the current state of that staff member's calendar and replaces the stored busy intervals, so a lost or out-of-order notification does no harm, and the daily full re-synchronisation is the safety net. Bookings are written to the calendar idempotently on the booking id. The contract is [ICR-AB-0002](../contracts/ICR-AB-0002-calendar-sync.md).

### Consequences

* Good, because there is no per-calendar fee, so the running cost does not climb as the product succeeds.
* Good, because no third party sits between the platform and staff calendars, which keeps the access review simple.
* Good, because changes arrive in seconds and the platform spends the providers' quotas on changes, not on looking.
* Bad, because the build and upkeep are ours: two provider-specific connectors, kept current as the providers change their APIs, scopes and limits. This is a cost in engineering time that a bought service would carry for us.
* Bad, because a third provider means a third connector, not a configuration change.
* Bad, because change subscriptions expire and must be renewed before they lapse; a missed renewal silently stops updates. This is the open issue in the contract, and needs a renewal schedule and an alert.

### Confirmation

| Action | Owner | Target date | Status |
| :--- | :--- | :--- | :--- |
| Build estimate for the two connectors, and a quote from at least one aggregation service for the expected number of calendars, compared before build starts | Lead back-end engineer | 2026-10-24 | not started |
| A calendar change appears in search within 60 seconds; after an hour's provider outage everything is consistent within ten minutes of its return (ICR-AB-0002 acceptance criteria 1 and 4) | Lead back-end engineer | 2026-11-14 | not started |
| Subscription renewal schedule designed, with an alert on a missed renewal | Lead back-end engineer | 2026-10-31 | not started |

## Pros and Cons of the Options

### Build on each provider's own API

* Good, because the running cost is flat and the access is the narrowest possible.
* Good, because the failure modes are the providers' own, and well documented.
* Bad, because the engineering time is ours, up front and for every provider API change after.

### Buy through an aggregation platform

* Good, because one vendor would cover many calendar systems and carry the upkeep.
* Bad, because the recurring cost grows with every calendar connected, and the vendor sets the price.
* Bad, because it adds a party that holds staff calendar grants, and its own delay on every change.
* Bad, because we need only two providers, which are already well served by their own APIs.

### Evaluation Summary

| Criterion | Build | Buy |
| :--- | :--- | :--- |
| Up-front cost | medium | low |
| Running cost as calendars grow | flat | grows with every calendar |
| Ongoing engineering effort | medium | low |
| Freshness | high | medium |
| Third party holds staff grants | no | yes |
| Vendor dependency | none | high |

## More Information

This is the library's [Integration Native Connectors (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-native-cloud) pattern: a supported connector to a SaaS provider's own interface, used directly, trading some portability for speed, control and cost. The decision turns on the number of providers: with two, building is small and bounded; with many, buying would be the better case, and this record should then be revisited.

### Governance

| | |
| :--- | :--- |
| Decision tier | B — a build-versus-buy choice with a lasting cost and a security review |
| Initiative and phase | Appointment Booking — design |
| Endorsed | 2026-10-02, architecture review |
| Deviates from principles | None |

### Related Decisions and Documents

| Reference | Relation | Description |
| :--- | :--- | :--- |
| [ADR-AB-0003 Availability source of truth](0003-availability-source-of-truth.md) | relates to | Why search reads a copy that this integration keeps current |
| [ICR-AB-0002 Calendar synchronisation](../contracts/ICR-AB-0002-calendar-sync.md) | reference | The contract, including its open issue on subscription renewal |
| [Availability requirements](../requirements/availability.md) | reference | The freshness the integration must achieve |
