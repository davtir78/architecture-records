---
template: architecture-decision-record
status: accepted
date: 2026-10-02
decision-makers: Solution architect; Platform engineer; Head of engineering
consulted: Security architect; Data architect; Finance partner
informed: Product owner; Support team
---

# ADR-AB-0006 Run on containers in one Australian region, recoverable into a second

## Context and Problem Statement

The booking platform serves many small businesses from one deployment. Its Booking API is promised at 99.9% availability a month, the data must stay in Australia, traffic is modest (a peak of 200 requests a second across all businesses, 50 for any one), and the team that runs it is small. What it costs to run, and what it costs when it fails, are both decided here: how much capacity is paid for when idle, and how much standby is paid for against a regional failure that may never happen.

Where and how does the platform run, and how is it recovered if its region is lost, at a cost proportionate to a small product?

## Decision Drivers

* **99.9% a month** for the Booking API ([ICR-AB-0001](../contracts/ICR-AB-0001-booking-api.md)).
* **Data stays in Australia.**
* **Pay for what is used:** traffic is low overnight and spiky on Monday mornings, so capacity that idles is money wasted.
* **A small team:** few moving parts, and a platform it can operate without a specialist.
* **Recovery proportionate to the risk:** losing a whole region is rare, and the business can tolerate an hour's outage and a few minutes of lost bookings, so a second region need not be paid for while it sits unused.
* **Not locked to one cloud's proprietary runtime** more than the product needs.

## Considered Options

* Containerised services on a managed container platform in one Australian region across three availability zones, with infrastructure as code that can rebuild the platform in a second region from backups
* Serverless functions for every service
* Virtual machines that we patch and scale ourselves
* Active-active in two Australian regions — not taken forward, because it roughly doubles the running cost and adds data-consistency problems that a one-hour recovery target does not need.

## Decision Outcome

Chosen option: "Containers in one region, recoverable into a second", because three availability zones in one region meet the 99.9% target, scaling containers on request rate and queue depth keeps idle cost low, and rebuilding in a second region from code and backups gives the recovery the business asked for without paying for a standby.

Services run as containers scaled on request rate, and the workers that feed on queues scale on queue depth. The database is a managed, multi-zone service with point-in-time recovery ([ADR-AB-0007](0007-data-store.md)). Recovery targets are a five-minute recovery point and a one-hour recovery time, rehearsed twice a year ([SAD section 8.2](../sad.md#82-disaster-recovery-dr--business-continuity)).

No dollar figure is given here. The estimate belongs in the business case; this record names what drives it: the baseline of the container platform, the managed database and its replicas, the data transfer between zones, and the backups.

### Consequences

* Good, because capacity follows load, so quiet hours cost little and Monday mornings are covered.
* Good, because no second region runs idle: the cost of resilience against a regional loss is the backups and the rehearsal, not a second environment.
* Good, because containers can move between clouds with far less rework than functions written for one provider.
* Bad, because losing the region means about an hour's outage and up to five minutes of lost writes. That is a business risk the product owner must accept knowingly, not an engineering detail.
* Bad, because a recovery rehearsed twice a year can rot between rehearsals; the test is the only evidence it works.
* Bad, because a container platform has a baseline cost even at zero traffic, which functions would not.
* Neutral, because the choice should be revisited if peak traffic stays far below the plan (functions might then cost less), or if the availability target rises beyond what one region can give (active-active would then be needed).

### Confirmation

| Action | Owner | Target date | Status |
| :--- | :--- | :--- | :--- |
| Restore into the second region and run the booking acceptance tests, twice a year (SAD section 8.2) | Platform engineer | 2026-12-15 | not started |
| Cost review against the estimate after the first full quarter in production | Finance partner | 2027-04-30 | not started |
| Product owner records acceptance of an hour's outage and five minutes' lost writes in a regional failure | Product owner | 2026-10-31 | not started |

## Pros and Cons of the Options

### Containers in one region, recoverable into a second

* Good, because it matches the availability target and the traffic without paying for unused standby.
* Good, because the recovery is code and backups, which are cheap to keep.
* Bad, because regional recovery is slow and lossy by design.

### Serverless functions for every service

* Good, because there is no idle compute to pay for.
* Bad, because cold starts press on the 400 ms search target, and the long-lived connections to the database and the workers that drain queues fit functions poorly.
* Bad, because the code ends up written for one provider's runtime.

### Virtual machines we manage

* Good, because the cost is predictable and the control is total.
* Bad, because patching, scaling and failover are ours, which a small team should not carry.

### Active-active in two regions

* Good, because a regional failure would barely be noticed.
* Bad, because it roughly doubles run cost and forces a decision about writing to two databases at once, for a risk the business has said it can tolerate.

### Evaluation Summary

| Criterion | Containers, one region + rebuild | Serverless | Virtual machines | Active-active |
| :--- | :--- | :--- | :--- | :--- |
| Run cost when idle | medium | low | high | high |
| Run cost at peak | medium | medium | medium | high |
| Operational effort | low | low | high | high |
| Survives a zone failure | yes | yes | with work | yes |
| Survives a region failure | slowly (about 1 hour) | slowly | slowly | yes |
| Portability | high | low | high | high |

## More Information

The recovery targets and the rehearsal are in the [SAD, section 8.2](../sad.md#82-disaster-recovery-dr--business-continuity). Messaging and calendar providers are outside this decision: their outages are handled by the design of the services that call them (see the [notification requirements](../requirements/notifications.md) and [ADR-AB-0004](0004-calendar-integration.md)).

### Governance

| | |
| :--- | :--- |
| Decision tier | A — fixes the running cost profile and the recovery risk the business carries |
| Initiative and phase | Appointment Booking — design |
| Endorsed | 2026-10-02, architecture review |
| Deviates from principles | None |

### Related Decisions and Documents

| Reference | Relation | Description |
| :--- | :--- | :--- |
| [ADR-AB-0007 Data store](0007-data-store.md) | depends on | The managed, multi-zone database with point-in-time recovery |
| [ICR-AB-0001 Booking API](../contracts/ICR-AB-0001-booking-api.md) | reference | The 99.9% availability and 50-requests-a-second limits this must meet |
| [Solution Architecture Design](../sad.md) | reference | Section 8, deployment and recovery |
