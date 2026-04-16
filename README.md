# Open Preference Initiative

Open standards for portable, user-controlled preference data.

## Status

OPI is in early draft. All specifications are at version 0.1 and are open for comment. No conformance claims against these documents are authoritative until a version 1.0 release.

## Specifications

| ID | Title | Version | Status |
|----|-------|---------|--------|
| [OPI-EPS](specs/OPI-EPS-0.1.md) | Extensible Preference Schema | 0.1 | Draft |
| [OPI-PD](specs/OPI-PD-0.1.md) | Preference Discoverability | 0.1 | Draft |
| [OPI-TRUST](specs/OPI-TRUST-0.1.md) | Trust Registry | 0.1 | Draft |

**OPI-EPS** defines the JSON bundle format, preference element structure, and versioning rules. It aligns with existing vocabularies (FHIR, BCP 47, ISO 4217, ISO 80000, schema.org) where those already cover the domain.

**OPI-PD** defines how a requestor discovers a subject's preference resource using WebFinger, and how preferences are retrieved under one of three access tiers: public, client, or user. User-tier access uses OAuth 2.0 Rich Authorization Requests or UMA 2.0.

**OPI-TRUST** defines a signed, subscription-based trust list format that preference servers use to evaluate the attributes of verified requestors. Multiple registries may coexist, and subscribers compose their results under local policy.

## Why this exists

Today, your preferences live inside opaque vendor profiles. You restate them every time you book a hotel, visit a new clinic, or set up a new app. Companies that do store preferences rarely let you inspect them, correct them, or carry them to a competitor. As automated agents start acting on our behalf, the cost of that fragmentation grows: every agent has to relearn the same facts, or worse, it guesses.

OPI's goal is to make preferences behave like email addresses: owned by the subject, portable between services, and interoperable by default.

## How to participate

OPI is a volunteer effort, and contributions are welcome from anyone.

- **Read a spec** and open a GitHub issue with questions, corrections, or disagreements.
- **Propose changes** by following the process in [CONTRIBUTING.md](CONTRIBUTING.md).
- **Report issues** using the templates in `.github/ISSUE_TEMPLATE/` for substantive concerns, editorial fixes, or open questions.
- **Implement** the specs experimentally in any language, and link your work back here.

## Governance

OPI is an open initiative with a light editorial process while at version 0.x. See [GOVERNANCE.md](GOVERNANCE.md) for how decisions are made and how contributors can take on editorial roles.

## License

Specifications and other documentation in this repository are licensed under the [Creative Commons Attribution 4.0 International License](LICENSE) (CC-BY-4.0). Future implementation code contributed to this repository will be licensed under Apache-2.0 unless noted otherwise.

## Contact

GitHub Issues on this repository are the primary channel for discussion, questions, and proposals.

Editor: Robin Monks.
