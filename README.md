# Architecture records

Markdown templates for architecture records, with a worked example of each, from [IT Architecture Patterns](https://www.itarchitecturepatterns.net).

A record is a short document, kept beside the system it describes, that says what was decided or agreed and why. Written in plain Markdown with a little front matter, records can be reviewed in a pull request, linked to one another, and checked by a script.

## Templates

| Template | What it records | File |
| :--- | :--- | :--- |
| Architecture decision record (ADR) | One significant decision: the options, the choice and its consequences | [templates/architecture-decision-record.md](templates/architecture-decision-record.md) |
| Solution architecture design (SAD) | The design of one system: stakeholders, context, components, runtime behaviour, data, security, deployment, risks | [templates/solution-architecture-design.md](templates/solution-architecture-design.md) |
| Reference architecture | The shared architecture of a whole domain: layers, product-neutral components, principles, roadmap | [templates/reference-architecture.md](templates/reference-architecture.md) |

Each template has a JSON Schema for its front matter in [templates/schema/](templates/schema/). The template page on the site shows each one beside its worked example and says how it relates to the work it builds on.

## Worked examples

| Example | What it shows |
| :--- | :--- |
| [Appointment Booking (sample)](samples/appointment-booking/README.md) | A fictional embeddable booking system documented end to end: a SAD, seven decisions, three requirements documents and three interface contracts, all linked to one another. It also has a [working demo](https://github.com/davtir78/appointment-booking-demo) that runs the design |
| [Integration domain](samples/integration-domain/README.md) | A reference architecture written out in the template's form |

## Using a template

Copy the template file, replace every `{…}` slot with your own content, and keep the numbered headings in order. A section that does not apply says "Not applicable" and why. The rules are in each template's comments.

## Credits

- The decision record template is based on [MADR](https://adr.github.io/madr/) 4.0.0, used under its dual licence, MIT or CC0-1.0. The evaluation, principle-alignment and governance sections are additions.
- The SAD template is shaped by the structure of [arc42](https://arc42.org) (Gernot Starke and Peter Hruschka, CC BY-SA 4.0) and the stakeholder-and-concern model of ISO/IEC/IEEE 42010. No text is copied from either.

## Where this comes from

These files are published from the source of the IT Architecture Patterns site, so changes are made there and copied here. To suggest one, open an issue on this repository.
