# Appointment Booking (sample)

A fictional but realistic system, documented through the templates on IT Architecture Patterns that describe a system. It exists to show what good records look like **and how they connect**: the requirements say what must be true, the decisions say how the design makes it true, the contract cites the decision, the design cites the pattern, and the design aligns to the domain's reference architecture.

> This is a teaching example. The businesses, staff and people in it are fictional, and nothing here describes a real product.

## The system

A business embeds a booking widget in its own website. Its customers pick a service, see the free times of the staff who offer it, and book, reschedule or cancel without phoning. Staff calendars stay in step with bookings, and customers get confirmations and reminders.

The worked examples use one fictional business, **Example Clinic (fictional)**: three staff (two physiotherapists and an exercise physiologist), three services, open on weekdays, with staff calendars in Google Calendar or Microsoft 365.

### Actors

| Actor | Wants to |
| :--- | :--- |
| **Customer** | Find a time that suits them and book it without an account or a phone call |
| **Staff member** | Keep one calendar, never be double-booked, see who is coming |
| **Business administrator** | Set services, staff, working hours and booking rules |
| **Website owner** | Add booking to their site with one snippet, without it breaking their page |

### In scope

- Searching availability by service, staff member and date, in the customer's time zone
- Booking, rescheduling and cancelling, with a hold while the customer completes the form
- Two-way sync with staff calendars in Google Calendar and Microsoft 365
- Confirmation and reminder messages by email and SMS
- Many businesses on one service, each isolated from the others

### Deliberately out of scope

Payments and deposits, waiting lists, group classes, staff rostering, and native mobile apps. Each is a reasonable next step, and leaving them out keeps the records focused on the decisions that shape the core.

## The records

| Record | Template | What it shows |
| :--- | :--- | :--- |
| [Solution Architecture Design](sad.md) | SAD | The whole system: stakeholders and quality goals, context, components, how it behaves at run time, integration, data, security, deployment, risks |
| [ADR-AB-0001 Embed the widget in an iframe](decisions/0001-embed-mechanism.md) | MADR | Isolation against integration, on a page the vendor doesn't control |
| [ADR-AB-0002 Expose booking as a public REST API behind a gateway](decisions/0002-api-exposure.md) | MADR | How the capability is reached from a browser on a site the vendor doesn't control, by anonymous customers |
| [ADR-AB-0003 Availability from a synchronised copy, checked live on confirm](decisions/0003-availability-source-of-truth.md) | MADR | Where the truth lives when it lives in someone else's calendar |
| [ADR-AB-0004 Build calendar connectors on each provider's own API, rather than buy](decisions/0004-calendar-integration.md) | MADR | A build-versus-buy choice whose cost grows with every calendar if bought |
| [ADR-AB-0005 Shared database with row-level tenant isolation](decisions/0005-multi-tenancy.md) | MADR | Many businesses, one service |
| [ADR-AB-0006 Run on containers in one Australian region, recoverable into a second](decisions/0006-hosting-and-recovery.md) | MADR | The run cost and the recovery risk the business carries |
| [ADR-AB-0007 Hold the data in a managed PostgreSQL database](decisions/0007-data-store.md) | MADR | A database that can enforce the guarantees itself |
| [Booking requirements](requirements/booking.md) | Requirements (OpenSpec form) | Booking, the no-overlap guarantee, holds, changing and cancelling, repeated requests |
| [Availability requirements](requirements/availability.md) | Requirements (OpenSpec form) | What is shown as free, how fresh, and times across zones and daylight saving |
| [Notification requirements](requirements/notifications.md) | Requirements (OpenSpec form) | What customers are told, and what must hold when delivery fails; the outbox is a design note here |
| [ICR-AB-0001 Booking API](contracts/ICR-AB-0001-booking-api.md) | ICR | Widget to Booking API: synchronous, public, personal information |
| [ICR-AB-0002 Calendar synchronisation](contracts/ICR-AB-0002-calendar-sync.md) | ICR | Change notifications from calendar providers |
| [ICR-AB-0003 Notification delivery](contracts/ICR-AB-0003-notifications.md) | ICR | Asynchronous messages to an email and SMS provider |

### Decision, requirement or design note?

Only **major** decisions are ADRs: ones that are hard or costly to reverse, that fix the structure or a major quality of the system, that commit real money or a vendor, or that touch more than one team. Everything else is recorded where it belongs:

- A **requirement** says what must be true, and can be checked: no two bookings overlap; times are right across daylight saving. It lives in [requirements/](requirements/), written with SHALL and WHEN/THEN scenarios.
- A **design note** records how a requirement is met when the choice is local and cheap to change: the hold and the overlap constraint, the UTC-instants design, the transactional outbox. It sits beside the requirement it serves.
- A **decision record** is for the choice a sensible team could have made differently and would pay dearly to undo: how the widget is delivered, how the API is exposed, who owns the availability data, build or buy, how tenants are isolated, where it runs and how it recovers, which database.

A decision's tier (A or B) in its governance table says how major it is. A tier C item is a design note, not an ADR.

The reference architecture example is not part of this system: a reference architecture describes a domain, so the template's worked example is the site's [Integration Reference Architecture](../integration-domain/reference-architecture.md), and this system's SAD shows how it aligns to it (section 4.3).

## Patterns from the library

| Where | Pattern |
| :--- | :--- |
| Widget → Booking API, across the public internet | [Integration API Management (External)](https://www.itarchitecturepatterns.net/patterns/int-api-external) |
| Booking events inside the platform | [Integration Event Streaming (Internal)](https://www.itarchitecturepatterns.net/patterns/int-esp-internal) |
| Calendar providers' own APIs and change notifications | [Integration Native Connectors (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-native-cloud) |
| Email and SMS provider behind a queue | [Integration Middleware Services (Cloud)](https://www.itarchitecturepatterns.net/patterns/int-middleware-cloud) |

## Not yet written

- **Pattern records (APR)** for the patterns above, once the Architecture Pattern Record format (#12) is reviewed.
- **The Strategy Modeler model** (users, use cases, logical, physical).
- **The clickable demo**, in its own repository `appointment-booking-demo`.
