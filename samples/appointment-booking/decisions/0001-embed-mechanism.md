---
template: architecture-decision-record
status: accepted
date: 2026-09-26
decision-makers: Solution architect; Lead front-end engineer
consulted: Security architect; Accessibility specialist
informed: Product owner; Support team
---

# ADR-AB-0001 Embed the booking widget in an iframe, loaded by a small script

## Context and Problem Statement

A business adds booking to its own website by pasting a snippet. That website is not ours: it runs its own scripts, styles and third-party tags, and it can change at any time. The widget collects a customer's name, email address and phone number.

How should the widget be embedded so that it works on any site, looks right, and keeps what the customer types away from everything else running on the page?

## Decision Drivers

* **Protect personal information.** Anything the customer types must not be readable by the host page's own scripts or its third-party tags.
* **Survive any host page.** A site's CSS, fonts and script errors must not break the widget, and the widget must not break the site.
* **One-snippet install** for a website owner with no developer.
* **Accessibility:** WCAG 2.2 AA inside the widget regardless of the host's own markup.
* **Fit the page:** the widget should resize to its content and accept the business's brand colour.

## Considered Options

* An iframe, loaded and sized by a small script
* A web component (a custom element) rendered into the host page
* A JavaScript SDK that writes the widget directly into the host page's DOM
* A link that opens a hosted booking page in a new tab — not taken forward as the main option, because every booking leaves the business's site; it stays as a fallback for pages that block scripts.

## Decision Outcome

Chosen option: "An iframe, loaded and sized by a small script", because it is the only option that meets the first driver. A same-origin script on the host page can read an iframe's document only if it shares the iframe's origin, which it doesn't; with a web component or an SDK, the form lives in the host's own DOM, where any script on the page can read it.

### Consequences

* Good, because customer details are isolated from the host page's scripts by the browser's same-origin policy, not by our code.
* Good, because the host's CSS and script errors can't reach the widget, so it looks and behaves the same on every site.
* Good, because accessibility is ours to get right inside the frame, independent of the host's markup.
* Bad, because the frame needs its height reported to the host page (a `postMessage` from the frame to the loader), or it scrolls inside itself.
* Bad, because deep styling by the host is limited to what we expose (brand colour, font family), which some website owners will want more of.
* Neutral, because a host page with a strict Content Security Policy must allow our frame's origin; the install guide says how.

### Confirmation

| Action | Owner | Target date | Status |
| :--- | :--- | :--- | :--- |
| Automated test: a script on a test host page cannot read the booking form's fields | Lead front-end engineer | 2026-10-10 | not started |
| Accessibility audit of the widget inside a host page (keyboard, screen reader, focus return on close) | Accessibility specialist | 2026-10-24 | not started |

## Pros and Cons of the Options

### An iframe, loaded and sized by a small script

The loader is a few kilobytes: it creates the frame, passes the business id and theme, and listens for height changes.

* Good, because the browser isolates the frame's content from the host page.
* Good, because the frame can set its own strict Content Security Policy.
* Neutral, because the loader must validate the origin of every message it receives.
* Bad, because resizing and focus management across the frame boundary need care.

### A web component rendered into the host page

* Good, because it can inherit the host's fonts and fit its layout naturally.
* Neutral, because shadow DOM keeps styles apart, but not scripts.
* Bad, because the form's inputs are in the host document, so any script there can read them.

### A JavaScript SDK writing into the host page's DOM

* Good, because it gives the host the most control over look and behaviour.
* Bad, because it has every weakness of the web component and none of shadow DOM's style isolation.
* Bad, because a host page's script error can stop the booking flow.

### Evaluation Summary

| Criterion | Iframe + loader | Web component | JavaScript SDK |
| :--- | :--- | :--- | :--- |
| Security risk | low | high | high |
| Robustness | high | medium | low |
| Implementation cost | medium | medium | low |
| Alignment with business outcomes | high | medium | medium |

### Alignment to Principles

| Principle | Iframe + loader | Web component | JavaScript SDK |
| :--- | :--- | :--- | :--- |
| Least privilege | aligned — the host page has no access to the form | not aligned | not aligned |
| Secure by default | aligned — isolation comes from the browser | partly | not aligned |

## More Information

This decision sets the trust boundary the rest of the design relies on: everything the widget sends crosses the public internet to the Booking API, which is why that integration follows [Integration API Management (External)](https://www.itarchitecturepatterns.net/patterns/int-api-external).

### Governance

| | |
| :--- | :--- |
| Decision tier | B — shapes the public interface and the handling of personal information |
| Initiative and phase | Appointment Booking — design |
| Endorsed | 2026-09-26, architecture review |
| Deviates from principles | None |

### Related Decisions and Documents

| Reference | Relation | Description |
| :--- | :--- | :--- |
| [ICR-AB-0001 Booking API](../contracts/ICR-AB-0001-booking-api.md) | relates to | The contract the widget calls |
| [ADR-AB-0002 API exposure](0002-api-exposure.md) | relates to | How the embedded widget reaches the platform |
| [Solution Architecture Design](../sad.md) | reference | Section 5, integration design |

### Review Feedback

| Stakeholder | Team / role | Feedback | Response |
| :--- | :--- | :--- | :--- |
| Security architect | Security | Every message between frame and loader must be origin-checked | Added to the loader's specification and to the confirmation test |
