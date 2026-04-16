# Governance

This document describes how decisions are made in the Open Preference Initiative (OPI) while the project is at version 0.x. It is intentionally lightweight. Governance will be revisited before any specification reaches version 1.0.

## Scope

This document covers:

1. Who can propose changes.
2. How changes are decided.
3. How editors are appointed.
4. How this governance document itself is changed.

## Roles

### Contributors

Anyone who opens an issue, comments on a pull request, or submits a pull request is a contributor. Contribution does not require formal affiliation.

### Editors

Editors are responsible for:

- Merging changes to specifications.
- Cutting version releases.
- Maintaining the issue tracker and assigning labels.
- Ensuring proposed changes are consistent across the three core specifications.

The initial editor is Robin Monks. Additional editors may be appointed by consensus of the existing editors when the project needs broader coverage or succession planning.

### Working group

At version 0.x there is no formal working group. A working group will be chartered before version 1.0 to take over editorial decisions and cross-specification coordination.

## Decision process

### Editorial changes

Editorial changes correct typos, clarify language, fix broken references, or reorganize content without altering normative requirements. Any editor may merge editorial changes after a brief review period (48 hours on the pull request for comment).

### Substantive changes

Substantive changes modify normative requirements (anything expressed with RFC 2119 keywords), add or remove specification members, or alter the behavior of a conforming implementation.

Substantive changes require:

1. An issue describing the problem and proposed direction.
2. A pull request implementing the change.
3. At least one editor approval.
4. A minimum open period of 7 days to allow public comment.
5. Resolution of any outstanding objections raised during the comment period. An objection is considered resolved when the objector either withdraws it, accepts the proposed response, or declines to engage further after a reminder.

If consensus cannot be reached, the editor may elect to defer the change, split it into smaller changes, or publish it as an experimental extension rather than incorporating it into a core specification.

### Versioning

The three core specifications (OPI-EPS, OPI-PD, OPI-TRUST) are versioned independently using Semantic Versioning 2.0.0.

- Patch releases: editorial fixes only.
- Minor releases: backward-compatible additions.
- Major releases: breaking changes.

Before version 1.0, minor and patch distinctions are advisory. The first stable release of each specification will be numbered 1.0.0.

## Changes to this document

Changes to this governance document follow the substantive change process above, with an extended minimum open period of 14 days.

## Conflicts of interest

Editors who have a material conflict of interest in a specific change (such as employment by an implementer whose commercial interest depends on the outcome) should disclose the conflict and defer to another editor for the final decision on that change.

## Dispute resolution

While OPI is at version 0.x, the initial editor has final authority over disputes. A more formal dispute resolution process will be defined as part of the version 1.0 working group charter.
