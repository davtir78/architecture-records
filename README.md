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

Copy the template file and replace every `{…}` slot with your own content. Each template has its own rule for sections you do not need:

- **Architecture decision record:** remove an optional element you do not use, as its comments say.
- **Solution architecture design:** keep the numbered headings in order, and write "Not applicable" and why under one that does not apply.
- **Reference architecture:** keep the headings, and fill each one.

## Credits

- The decision record template is based on [MADR](https://adr.github.io/madr/) 4.0.0, which is licensed MIT or CC0-1.0, and is used here under its CC0-1.0 option. The evaluation, principle-alignment and governance sections are additions.
- The SAD template is shaped by the structure of [arc42](https://arc42.org) (Gernot Starke and Peter Hruschka, CC BY-SA 4.0) and the stakeholder-and-concern model of ISO/IEC/IEEE 42010. No text is copied from either.

## Where this comes from

These files are published from the source of the IT Architecture Patterns site, so changes are made there and copied here. To suggest one, open an issue on this repository.

## Licence

The templates, schemas and worked examples are licensed under [Creative Commons Attribution 4.0 International](LICENSE) (CC BY 4.0): you may copy, adapt and use them, including commercially, if you give credit to IT Architecture Patterns and say what you changed. The decision record template is also used under MADR's CC0-1.0 option, as credited above.
