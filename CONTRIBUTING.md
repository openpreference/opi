# Contributing to the Open Preference Initiative

Thank you for your interest in contributing. This document explains how to propose changes to the Open Preference Initiative specifications and related materials.

## Before you contribute

1. Read [the README](README.md) to understand the scope of the project.
2. Read [GOVERNANCE.md](GOVERNANCE.md) to understand how decisions are made.
3. Search [existing issues](https://github.com/openpreference/opi/issues) to see if your topic has already been raised.

## Types of contribution

### Issues

Open an issue for:

- Questions about a specification.
- Reported errors, ambiguities, or contradictions in the text.
- Proposed substantive changes you want to discuss before writing a pull request.
- Requests for new preference types or verification categories.

Use the issue templates in `.github/ISSUE_TEMPLATE/` to ensure your issue has the information reviewers need.

### Pull requests

Open a pull request for:

- Editorial fixes (typos, clarity, broken links).
- Implementation of a change that has been discussed in an issue.
- Documentation improvements outside the specifications themselves (README, governance, contributing, and similar).

### Implementations

Experimental implementations in any programming language are welcome. Implementations do not need to live in this repository. If you build one, open an issue linking to it and we will add it to an implementations list.

## Change classes

Changes fall into one of two classes. Which class a change belongs to affects the review process.

### Editorial

Editorial changes correct language without changing what a conforming implementation must do. Examples:

- Fixing a typo.
- Rewording a sentence for clarity.
- Repairing a broken cross-reference.
- Reordering paragraphs.

Editorial changes are reviewed quickly and merged after a short comment window (48 hours).

### Substantive

Substantive changes alter normative requirements, meaning anything expressed with RFC 2119 keywords (MUST, SHOULD, MAY, and so on), or anything that changes the observable behavior of a conforming implementation.

Substantive changes require:

1. An issue describing the problem and proposed direction.
2. A pull request implementing the change, linked to the issue.
3. A minimum open period of 7 days for public comment.
4. At least one editor approval.

Details are in [GOVERNANCE.md](GOVERNANCE.md).

## Style

### Specification text

- Use RFC 2119 keywords (MUST, MUST NOT, REQUIRED, SHALL, SHOULD, SHOULD NOT, MAY, OPTIONAL) in all capitals for normative statements.
- Lowercase uses of those words are not normative.
- Cite RFCs, ISO standards, and existing vocabularies by bracketed short name (for example `[RFC 7515]`, `[FHIR R4B]`) and list them in the References section.
- Use examples marked "non-normative" when the example is illustrative rather than required.
- Keep sections numbered consistently across the three core specifications.

### Prose

- Write in plain English. Prefer short sentences to long ones.
- Do not use em-dashes (U+2014). Use commas, parentheses, colons, or sentence breaks instead.
- Use the Oxford comma.
- Use "requestor" (not "requester") to match the existing specification text.

### Markdown

- Use ATX-style headers (`# Heading`, not underlines).
- Use fenced code blocks with a language tag.
- Use tables for member definitions (name, type, required, description).

## Pull request workflow

1. Fork the repository.
2. Create a branch from `main` with a descriptive name (for example `fix/typo-in-eps-section-5` or `propose/add-industry-category`).
3. Make your changes. Keep commits focused.
4. Open a pull request using the PR template.
5. Link any related issues.
6. Respond to review feedback.
7. An editor will merge the pull request once review and, for substantive changes, the comment period are complete.

## Sign-off and licensing

By submitting a contribution, you agree that your contribution is licensed under the terms described in [LICENSE](LICENSE). Documentation contributions are licensed under Creative Commons Attribution 4.0 International (CC BY 4.0). Any source code contributions are licensed under the Apache License 2.0 unless noted otherwise in the relevant file.

You do not need to sign a separate contributor license agreement. Opening a pull request is taken as agreement with the licensing terms of this repository.

## Security

If you believe you have found a security issue in a specification (for example, a requirement that would enable an attack on a conforming implementation), please open an issue and label it `security`. If you believe the issue is sensitive enough that public disclosure would cause harm before a fix is available, contact the editor directly rather than opening a public issue.

## Questions

If you are not sure whether something belongs as an issue or a pull request, or you have a question about the specifications, open an issue using the "Question" template.
