---
template: architecture-decision-record
status: accepted
date: 2026-10-02
decision-makers: Solution architect; Lead back-end engineer
consulted: Security architect; Lead front-end engineer
informed: Product owner; Support team
---

# ADR-AB-0002 Expose booking as a public REST API behind a gateway, identified by a widget key

## Context and Problem Statement

The widget runs in a customer's browser, inside an iframe on a website the vendor does not control ([ADR-AB-0001](0001-embed-mechanism.md)). To show free times and to hold, confirm, change and cancel bookings it must call the booking platform. The platform's capability may later be reached by other callers too, such as a business's own tools.

How is the booking capability exposed, so that a browser on any registered website can use it, an anonymous customer can book without an account, a public endpoint resists abuse, and the interface can change without breaking the widgets already installed on other people's sites?

## Decision Drivers

* **Callable from a browser** on a third-party site, with no server work for a small business that has no developer.
* **Anonymous customers:** booking needs no account, so there is no per-customer login to authenticate with.
* **A public endpoint will be abused:** it needs origin control, rate limits and a way to shut off one business without affecting the others.
* **An installed widget cannot be recalled:** the interface must be stable and versioned, with notice before it changes.
* **Low cost and complexity** for a small team to run.

## Considered Options

* A public REST API behind an API gateway, identified by a public widget key restricted to the domains the business registers, with the major version in the path
* A GraphQL endpoint with the same key
* A proxy hosted by each business: its server calls the platform with a secret key, and the widget calls that proxy
* OAuth sign-in for every customer — not taken forward, because customers book anonymously (no account is a requirement)

## Decision Outcome

Chosen option: "A public REST API behind an API gateway, identified by a widget key", because it works from any registered site without a server on the business's side, maps directly onto the resources a booking is made of (services, availability, holds, bookings), and puts every cross-cutting control in one place, the gateway.

Each request carries the business's public widget key, which identifies exactly one business. The gateway checks the calling domain against those the business registered, applies a rate limit per business, and routes to the services. Writes carry an idempotency key so a repeated request is safe. The contract, with its errors and limits, is [ICR-AB-0001](../contracts/ICR-AB-0001-booking-api.md).

### Consequences

* Good, because a business installs one snippet and runs nothing.
* Good, because the gateway is the single place for origin checks, rate limits, tracing and shutting off a key.
* Good, because REST's resources, status codes and cacheable reads suit the operations, and the contract is easy to describe and test.
* Bad, because the widget key is public: it identifies a business but proves nothing about the caller. Abuse control therefore rests on registered origins, rate limits and bot protection, none of which is authentication.
* Bad, because a booking takes several calls (search, hold, confirm), where a more specialised interface could take fewer.
* Bad, because a version in the path means supporting an old version for as long as widgets that use it remain installed; the contract promises 90 days' notice before a breaking change.
* Neutral, because a different style such as GraphQL could be added later behind the same gateway without changing this decision.

### Confirmation

| Action | Owner | Target date | Status |
| :--- | :--- | :--- | :--- |
| A request from an unregistered domain receives `403`; a repeated `POST` with the same key creates nothing new (ICR-AB-0001 acceptance criteria 3 and 4) | Lead back-end engineer | 2026-10-17 | not started |
| Bot protection on hold and confirm reviewed by the security architect before launch | Security architect | 2026-10-31 | not started |

## Pros and Cons of the Options

### Public REST API behind a gateway, identified by a widget key

* Good, because it works from any registered site with nothing for the business to host.
* Good, because the gateway centralises the controls a public endpoint needs.
* Bad, because the key is a public identifier, not a secret.

### GraphQL endpoint

* Good, because the widget could fetch exactly what a screen needs in one call.
* Bad, because rate limiting by cost, caching and abuse control are harder on a single flexible endpoint, and the widget's few screens do not need the flexibility.
* Bad, because the team has less experience of running it.

### A proxy hosted by each business

* Good, because the platform's key stays secret on the business's server.
* Bad, because it requires a developer and a server for every business, which defeats the one-snippet install.
* Bad, because each proxy is a place for a business to get the contract wrong.

### Evaluation Summary

| Criterion | Public REST + gateway | GraphQL | Business-hosted proxy |
| :--- | :--- | :--- | :--- |
| Works with no work by the business | yes | yes | no |
| Abuse control | medium | low | high |
| Operational risk | low | medium | high |
| Delivery time | low | medium | high |

## More Information

The gateway is built on the library's [Integration API Management (External)](https://www.itarchitecturepatterns.net/patterns/int-api-external) pattern, which is why the controls above are the ones that pattern lists for crossing a trust boundary.

### Governance

| | |
| :--- | :--- |
| Decision tier | A — fixes the shape of a public interface that is hard to change once widgets are installed |
| Initiative and phase | Appointment Booking — design |
| Endorsed | 2026-10-02, architecture review |
| Deviates from principles | None |

### Related Decisions and Documents

| Reference | Relation | Description |
| :--- | :--- | :--- |
| [ADR-AB-0001 Embed mechanism](0001-embed-mechanism.md) | depends on | Why the caller is a browser on a site the vendor does not control |
| [ADR-AB-0005 Multi-tenancy](0005-multi-tenancy.md) | relates to | The key resolves to one business, and every query is scoped to it |
| [ICR-AB-0001 Booking API](../contracts/ICR-AB-0001-booking-api.md) | reference | The contract this decision commits to |
| [Booking requirements](../requirements/booking.md) | reference | What the interface must make possible |
