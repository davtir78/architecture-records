# Integration domain: the reference architecture example

The worked example for the **Reference Architecture template** is this site's own [Integration Reference Architecture](reference-architecture.md), written out in the template's form. A reference architecture describes a domain rather than one system, so it isn't a record from the [Appointment Booking sample](../appointment-booking/README.md); that system's SAD shows how it aligns to this architecture instead.

`reference-architecture.md` is **generated** by `scripts/generate-ra-example.ts`, so don't edit it by hand:

- **From the site's data:** every component table (description and example products) and the patterns table, so they match what the site shows. `scripts/tests/ra-example.test.ts` fails if they drift.
- **Written in the script, from what the site already says:** the overview (the live article's own text), scope, principles, key decisions, the note on roadmap and lifecycle, and the change history. Their wording is for the owner to review.
