---
title: "OPI-EPS 0.1: Extensible Preference Schema"
doc_id: OPI-EPS
version: 0.1
status: draft
editor: Robin Monks
issued: 2026-04-16
tags: [open-preferences, opi, eps, draft]
---

# OPI-EPS 0.1: Extensible Preference Schema

## Abstract

This document defines the Extensible Preference Schema (EPS), a JSON-based format for representing versioned, machine-interpretable user preferences. EPS is one of the foundational specifications of the Open Preference Initiative (OPI). It defines only the shape of preference data; discovery and retrieval are specified in [OPI-PD], and requestor verification in [OPI-TRUST].

## Status of This Document

This is a draft at version 0.1. It is open for comment and subject to substantive change. No conformance claims against this document are authoritative until a version 1.0 release.

## 1. Introduction

Current systems store user preferences in opaque, vendor-specific formats that cannot be reused across services. EPS defines a common, versioned representation for preferences that can be produced and consumed by any conforming system, reusing existing domain standards where they apply.

## 2. Terminology

**Preference.** A typed statement about a subject's choice, expressed as a `type`, `version`, and `value`.

**Preference bundle.** A collection of preferences for a single subject, carried in a single envelope.

**Subject.** The entity a preference bundle describes. Expressed as a URI, typically an `acct:` URI per [RFC 7565] or an `https:` URI identifying a domain.

**Issuer.** The entity that produced and signed a preference bundle.

**Type URI.** An HTTPS URI that uniquely identifies a preference type. The URI SHOULD be dereferenceable to a human-readable or machine-readable definition.

## 3. Conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHOULD, SHOULD NOT, MAY, and OPTIONAL in this document are to be interpreted as described in [RFC 2119] and [RFC 8174] when, and only when, they appear in all capitals.

A conforming EPS document MUST validate against the bundle structure defined in Section 4. A conforming EPS producer MUST emit documents that validate. A conforming EPS consumer MUST reject documents that do not validate.

## 4. Preference Bundle

An EPS preference bundle is a JSON object with the following members.

| Member | Type | Required | Description |
|--------|------|----------|-------------|
| `schema` | string | yes | Absolute URI identifying the EPS schema version. |
| `subject` | string | yes | URI identifying the subject of the bundle. |
| `issued` | string | yes | RFC 3339 timestamp of bundle generation. |
| `expires` | string | no | RFC 3339 timestamp after which the bundle is no longer valid. |
| `preferences` | array | yes | Array of preference elements (Section 5). |

The `schema` member for this version MUST be `https://openpreference.org/eps/0.1`.

## 5. Preference Element

A preference element is a JSON object with the following members.

| Member | Type | Required | Description |
|--------|------|----------|-------------|
| `type` | string | yes | Absolute URI identifying the preference type. |
| `version` | string | yes | Semantic version of the element definition, per [SemVer]. |
| `value` | any | yes | The preference value. Type determined by the element definition referenced by `type`. |
| `source` | string | no | URI identifying the origin of this value, for provenance tracking. |
| `updated` | string | no | RFC 3339 timestamp of last value change. |

## 6. Value Types

Element definitions MAY specify values of any JSON type. Producers SHOULD use existing controlled vocabularies where applicable (Section 7). Element definitions MUST document the expected value type and any enumerations.

## 7. Baseline Alignments

Implementations SHOULD adopt existing vocabularies as the baseline for preference types in the following domains.

### 7.1 Healthcare preferences

Preferences in healthcare domains SHOULD map to resources defined in [FHIR R4B] or later, including but not limited to `Patient.communication`, `Patient.generalPractitioner`, and `AllergyIntolerance`.

### 7.2 Locale, unit, time, and currency preferences

Locale preferences SHOULD use [BCP 47] language tags. Unit preferences SHOULD use [ISO 80000] identifiers. Currency preferences SHOULD use [ISO 4217] codes. Time-zone preferences SHOULD use identifiers from the [IANA Time Zone] database.

### 7.3 Contact and communication preferences

Contact preferences SHOULD map to types defined in [schema.org ContactPoint] where applicable.

## 8. Versioning

### 8.1 Schema version

The `schema` member identifies the major version of the EPS document format. Minor and patch revisions to the format MUST NOT change the URI if they remain backward-compatible for consumers. Breaking changes MUST change the URI.

### 8.2 Element version

Each element's `version` member identifies the version of the referenced type definition. Element definitions MUST follow [SemVer]. Producers MUST emit the highest version they support for a given type. Consumers MUST accept any version at or below the latest version they support, and MAY ignore elements whose version they do not recognize.

## 9. Signing

### 9.1 JWS envelope

EPS bundles transmitted over untrusted channels MUST be wrapped in a JSON Web Signature envelope per [RFC 7515]. The JWS payload is the serialized EPS bundle.

### 9.2 Replay protection

Signed bundles MUST include both the `issued` member and an `expires` member. Consumers MUST reject bundles where `expires` is in the past or `issued` is further in the future than a locally-configured skew tolerance.

### 9.3 Key distribution

Public keys used to verify EPS signatures MUST be distributable as a JWK Set per [RFC 7517] referenced from the discovery document defined in [OPI-PD].

## 10. Examples (non-normative)

### 10.1 Minimal bundle

```json
{
  "schema": "https://openpreference.org/eps/0.1",
  "subject": "acct:alice@example.com",
  "issued": "2026-04-16T14:30:00Z",
  "expires": "2026-04-16T15:30:00Z",
  "preferences": [
    {
      "type": "https://openpreference.org/eps/0.1/types/locale",
      "version": "1.0.0",
      "value": "en-CA"
    }
  ]
}
```

### 10.2 Multi-domain bundle

```json
{
  "schema": "https://openpreference.org/eps/0.1",
  "subject": "acct:alice@example.com",
  "issued": "2026-04-16T14:30:00Z",
  "expires": "2026-04-16T15:30:00Z",
  "preferences": [
    {
      "type": "https://openpreference.org/eps/0.1/types/locale",
      "version": "1.0.0",
      "value": "en-CA"
    },
    {
      "type": "https://openpreference.org/eps/0.1/types/dietary/restrictions",
      "version": "1.0.0",
      "value": ["vegetarian"]
    },
    {
      "type": "https://openpreference.org/eps/0.1/types/currency",
      "version": "1.0.0",
      "value": "CAD"
    }
  ]
}
```

## 11. Security Considerations

Bundles carry personal data and MUST be transmitted over TLS. Producers MUST sign bundles delivered to any third party. Consumers MUST verify signatures and reject expired bundles.

## 12. Privacy Considerations

EPS does not specify access control; that is the responsibility of [OPI-PD]. Producers MUST NOT include preference elements outside the scope authorized for the requestor. Bundles SHOULD contain only the elements necessary for the requestor's stated purpose.

## 13. References

### 13.1 Normative

- [RFC 2119] Key words for use in RFCs to Indicate Requirement Levels
- [RFC 3339] Date and Time on the Internet: Timestamps
- [RFC 7515] JSON Web Signature (JWS)
- [RFC 7517] JSON Web Key (JWK)
- [RFC 7565] The 'acct' URI Scheme
- [RFC 8174] Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words
- [SemVer] Semantic Versioning 2.0.0

### 13.2 Informative

- [OPI-PD] Open Preference Initiative: Preference Discoverability, version 0.1
- [OPI-TRUST] Open Preference Initiative: Trust Registry, version 0.1
- [FHIR R4B] HL7 FHIR Release 4B
- [BCP 47] Tags for Identifying Languages
- [ISO 4217] Codes for the representation of currencies
- [ISO 80000] Quantities and units
- [IANA Time Zone] IANA Time Zone Database
- [schema.org ContactPoint] https://schema.org/ContactPoint
