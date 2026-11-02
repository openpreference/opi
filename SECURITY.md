# Security policy

## Reporting a vulnerability

Email **security@openpreference.org**. Please do not open a public GitHub issue for a security problem.

This covers two kinds of report:

- **Specification flaws.** A weakness in OPI-EPS, OPI-PD or OPI-TRUST as written, for example a signing, discovery, authorization or enumeration-resistance requirement that an attacker could get around even in a correct implementation.
- **The reference implementation at openpreference.org.** Bugs in the hosted server, its OAuth endpoints, its signed bundles or its trust list.

Include what you found, how to reproduce it, and what an attacker gains. A proof of concept helps but is not required.

## What happens next

- The editor acknowledges each report. OPI is run by volunteers, so allow a few days.
- Specification flaws are discussed privately until a fix is drafted, then published as a change under [CONTRIBUTING.md](CONTRIBUTING.md) with the `security` label. You are credited in the commit and the change summary unless you ask not to be.
- Implementation bugs are fixed on openpreference.org first and disclosed after the fix is live.

## Scope

The specifications are drafts at version 0.1. Reports about the drafts are welcome and taken seriously, but no version is yet suitable for production conformance claims.

Machine-readable contact details are at <https://openpreference.org/.well-known/security.txt>.
