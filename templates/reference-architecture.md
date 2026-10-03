---
template: reference-architecture
owner: "{name / role}"
status: "{draft | in review | endorsed}"
version: "{1.0}"
date: "{YYYY-MM-DD}"
applies-to: "{which systems, teams or programmes must follow this architecture}"
---

# {Domain} Reference Architecture

## Overview

{Two or three sentences: what capability this architecture provides, and for whom. Name the problem it solves rather than the products it uses.}

{What organises the model — for example the path data travels, the request flow, or the assets being protected — so readers know how to read the layers below.}

### Scope

* **In scope:** {the capabilities, platforms and boundaries this architecture covers}
* **Out of scope:** {what it deliberately does not cover, and where that is addressed instead}

### Principles

| Principle | What it means here |
| :--- | :--- |
| {e.g., API First} | {how the principle shapes decisions in this domain} |
| {e.g., Least Privilege} | {…} |

## Component Model Layers

{A layer groups components that serve the same purpose. Describe each component as a capability — what it does — and keep products in the examples column, so the model stays valid when a product changes.}

### 1. {Layer Name}

{One or two sentences on what this layer is responsible for.}

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **{Component Name}** | {What the component does and why it exists, independent of any product.} | {Product or service (AWS); Product or service (Azure); Product or service (GCP); Open-source option} |
| **{Component Name}** | {…} | {…} |

### 2. {Layer Name}

{…}

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **{Component Name}** | {…} | {…} |

## Cross-Cutting Layers

{Capabilities that apply across every layer above, such as governance, security, operations and delivery.}

### {Cross-Cutting Layer Name}

{…}

| COMPONENT | DESCRIPTION | EXAMPLES |
| :--- | :--- | :--- |
| **{Component Name}** | {…} | {…} |

## From Architecture to Patterns

{The reference architecture names the capabilities; patterns show how they are built. Link the patterns that implement each component so teams can move from "what" to "how".}

| COMPONENT | PATTERN | WHAT IT SHOWS |
| :--- | :--- | :--- |
| **{Component Name}** | {Pattern name and link} | {The design decision the pattern captures} |

## Key Decisions

{Decisions that shaped this architecture, recorded as ADRs.}

| Decision | Status | Record |
| :--- | :--- | :--- |
| {e.g., Standardise on one API gateway} | {accepted} | {link to ADR} |

## Roadmap and Lifecycle

{Where the organisation is moving from and to. Classify the products in use so investment follows the target state.}

| Component | Product | Lifecycle | Notes |
| :--- | :--- | :--- | :--- |
| **{Component Name}** | {Product} | {strategic / emerging / tactical / contain / decommission} | {…} |

## Change History

| Version | Date | Author | Change |
| :--- | :--- | :--- | :--- |
| {1.0} | {YYYY-MM-DD} | {name} | {initial version} |
