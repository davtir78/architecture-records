---
template: architecture-decision-record
status: accepted
date: 2026-09-26
decision-makers: Solution architect; Integration architect
consulted: Lead back-end engineer; Product owner
informed: Support team
---

# ADR-AB-0003 Show availability from a synchronised copy, and check it live on confirm

## Context and Problem Statement

A staff member's free time depends on two things: the working hours the business sets in our system, and everything else in their own calendar in Google Calendar or Microsoft 365 — a dentist appointment, a team meeting, leave. The second is not ours. It lives in someone else's service, changes without telling us unless we subscribe, and is subject to that provider's rate limits.

When a customer opens the widget, where does the answer to "when is Sam free next Tuesday?" come from?

## Decision Drivers

* **Search must be fast:** the widget shows a week of slots for several staff members at once, within a second.
* **A booking must never land on a real conflict**, including one added to the calendar a minute ago.
* **Provider rate limits:** calendar APIs cap requests per user and per application; a busy business page can't cost one call per slot.
* **Resilience:** a provider outage should not stop customers seeing availability.

## Considered Options

* A synchronised copy of busy times, kept current by the providers' change notifications, and checked live when a booking is confirmed
* Ask the calendar provider live on every search
* Only our own store: staff record every commitment in our system instead of their calendar — not taken forward, because staff will keep using their own calendar, and the store would quietly go stale.

## Decision Outcome

Chosen option: "A synchronised copy, checked live on confirm", because it is the only option that is fast enough for search and still keeps a booking from landing on a conflict the copy has not yet seen. Search reads our copy; confirmation re-reads the one staff member's calendar for the one slot, a single cheap request at the moment it matters.

When the live check cannot be made because the provider is down, the booking is confirmed anyway, the skipped check is logged, and the calendar is reconciled when the provider returns. The alternative, refusing the booking, is rejected below.

### Consequences

* Good, because search is served from our own store, so it is fast and doesn't spend provider quota.
* Good, because the live check at confirmation closes the gap between a calendar change and our copy catching up.
* Good, because a provider outage degrades search to "possibly stale" rather than "unavailable".
* Bad, because a customer can occasionally choose a slot that the live check then rejects; the widget must offer the next free slots immediately, not an error.
* Bad, because during a provider outage a booking can land on a calendar event the copy had not seen. That is a double booking the business must sort out by hand. It is accepted deliberately, because the other choice, refusing bookings while a third party is down, hands that party control over our availability.
* Bad, because it creates a synchronisation component to run and monitor — [ICR-AB-0002](../contracts/ICR-AB-0002-calendar-sync.md) is its contract.
* Neutral, because notification subscriptions expire and must be renewed on a schedule.

### Confirmation

| Action | Owner | Target date | Status |
| :--- | :--- | :--- | :--- |
| Metric and alert: age of the oldest unsynchronised calendar change, per business | Integration architect | 2026-10-17 | not started |
| Test: a conflict added to a staff calendar after search is rejected at confirmation | Lead back-end engineer | 2026-10-17 | not started |
| Product owner records acceptance of a possible double booking during a calendar provider outage, and the rate at which it is tolerated | Product owner | 2026-10-24 | not started |
| Alert on every skipped live check, so the business can look at the bookings made while it was skipped | Integration architect | 2026-10-24 | not started |

## Pros and Cons of the Options

### A synchronised copy, checked live on confirm

* Good, because it separates the high-volume read (search) from the high-stakes write (confirm).
* Neutral, because it depends on change notifications, with a periodic full re-sync as the safety net.
* Bad, because there are two paths to keep consistent.

### The same, but refuse the booking when the live check cannot be made

* Good, because no double booking can ever result from a stale copy.
* Bad, because every calendar provider outage stops bookings for every business that uses it, which turns a third party's availability into ours. The harm it prevents (an occasional double booking a person can resolve) is smaller than the harm it causes (no bookings at all).

### Ask the calendar provider live on every search

* Good, because there is nothing to keep in sync.
* Bad, because one widget view becomes dozens of provider calls, which runs into rate limits on any busy business page.
* Bad, because search is only as available and as fast as the slowest provider.

### Evaluation Summary

| Criterion | Synchronised copy + live check | Live on every search |
| :--- | :--- | :--- |
| Performance | high | low |
| Robustness | high | low |
| Operational cost | medium | high (quota) |
| Implementation cost | medium | low |

## More Information

The copy holds **busy intervals only** — start, end and whether a slot is blocked — never the titles or attendees of staff members' own appointments. That is the least information the decision needs, and it keeps other people's calendar contents out of our system.

### Governance

| | |
| :--- | :--- |
| Decision tier | B — determines an external dependency and a data flow |
| Initiative and phase | Appointment Booking — design |
| Endorsed | 2026-09-26, architecture review |
| Deviates from principles | None |

### Related Decisions and Documents

| Reference | Relation | Description |
| :--- | :--- | :--- |
| [Booking requirements](../requirements/booking.md) | relates to | The live check runs when a hold becomes a booking |
| [ADR-AB-0004 Calendar integration](0004-calendar-integration.md) | relates to | How the synchronised copy is kept current |
| [ICR-AB-0002 Calendar synchronisation](../contracts/ICR-AB-0002-calendar-sync.md) | depends on | The contract with the calendar providers |
| [Integration Native Connectors (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-native-cloud) | reference | The pattern the synchronisation follows |
