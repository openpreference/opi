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
| `iss` | string | yes | HTTPS origin of the preference server that produced the bundle, per [RFC 7519] Section 4.1.1. |
| `sub` | string | yes | URI identifying the subject of the bundle, per [RFC 7519] Section 4.1.2. |
| `iat` | number | yes | NumericDate of bundle generation, per [RFC 7519] Section 4.1.6. |
| `exp` | number | yes | NumericDate after which the bundle is no longer valid, per [RFC 7519] Section 4.1.4. |
| `preferences` | array | yes | Array of preference elements (Section 5). |

A preference bundle is a JWT Claims Set per [RFC 7519]. `schema` and `preferences` are private claim names per [RFC 7519] Section 4.3.

The `schema` member for this version MUST be `https://openpreference.org/eps/0.1`.

## 5. Preference Element

A preference element is a JSON object with the following members.

| Member | Type | Required | Description |
|--------|------|----------|-------------|
| `type` | string | yes | Absolute URI identifying the preference type. |
| `version` | string | yes | Semantic version of the element definition, per [SemVer]. |
| `value` | any | yes | The preference value. Type determined by the element definition referenced by `type`. |
| `source` | string | no | URI identifying the origin of this value, for provenance tracking. |
| `updated` | string | no | RFC 3339 timestamp of the last change to `value`. This member is preference data rather than a JWT claim, so it uses RFC 3339 rather than NumericDate. |

## 6. Value Types

Element definitions MAY specify values of any JSON type. Producers SHOULD use existing controlled vocabularies where applicable (Section 7). Element definitions MUST document the expected value type and any enumerations.

## 7. Baseline Alignments

Implementations SHOULD adopt existing vocabularies as the baseline for preference types in the following domains.

### 7.1 Healthcare preferences

Preferences in healthcare domains SHOULD map to resources defined in [FHIR R4B] or later, including but not limited to `Patient.communication`, `Patient.generalPractitioner`, and `AllergyIntolerance`.

### 7.2 Locale, unit, time, and currency preferences

Locale preferences SHOULD use [BCP 47] language tags. Unit preferences SHOULD use [UCUM] codes, using the case-sensitive variant. [ISO 80000] remains the authority for the definitions of quantities and units; [UCUM] supplies the machine-readable codes for them. Currency preferences SHOULD use [ISO 4217] codes. Time-zone preferences SHOULD use identifiers from the [IANA Time Zone] database.

### 7.3 Contact and communication preferences

Contact preferences SHOULD map to types defined in [schema.org ContactPoint] where applicable.

## 8. Versioning

### 8.1 Schema version

The `schema` member carries the major and minor version of this document, and changes with each published minor or major version. Patch versions of this document do not change the `schema` URI.

Before version 1.0, each minor version is a distinct format and consumers MUST NOT assume compatibility between `schema` URIs. From version 1.0, minor versions MUST be backward compatible: a consumer that supports `https://openpreference.org/eps/1.N` MUST accept any bundle whose `schema` is `https://openpreference.org/eps/1.M` where M is less than or equal to N, and MUST ignore members it does not recognize. Major versions are not compatible with one another.

### 8.2 Element version

Each element's `version` member identifies the version of the type definition referenced by `type`. Type definitions MUST be versioned per [SemVer]: a major version change indicates an incompatible change to the value type or semantics, a minor version change adds optional structure without breaking existing consumers, and a patch version change is editorial.

Producers MUST emit the highest version they support for a given type. A consumer MUST accept an element when its major version matches a major version the consumer supports for that type and its minor and patch versions are at or below the highest the consumer supports. A consumer MAY accept a higher minor version of a supported major version and MUST ignore any structure it does not recognize. A consumer MUST ignore an element whose major version it does not support, and MUST NOT reject the bundle on that account.

## 9. Signing

### 9.1 JWT envelope

A signed bundle is a JWT per [RFC 7519] whose Claims Set is the preference bundle. It MUST use JWS compact serialization per [RFC 7515] Section 7.1. The JOSE header MUST include `alg` (which MUST NOT be `none`), `kid`, and `typ` with the value `opi-eps+jwt` per [RFC 8725] Section 3.11.

Producers MUST use an asymmetric algorithm; symmetric `alg` values are prohibited because verification keys are published (Section 9.3). Producers MUST support `ES256` per [RFC 7518]. Consumers MUST support `ES256` and MAY support `EdDSA` per [RFC 8037]. Consumers MUST reject bundles using any `alg` they do not support.

### 9.2 Replay protection

Consumers MUST reject bundles whose `exp` is in the past or whose `iat` is further in the future than a locally configured skew tolerance.

### 9.3 Key distribution

Public keys used to verify EPS signatures MUST be published as a JWK Set per [RFC 7517] at the `https://openpreference.org/rel/keys` link in the discovery document defined in [OPI-PD]. Consumers MUST reject a bundle whose `kid` does not identify a key in that JWK Set.

### 9.4 Media types

A signed bundle is identified by the media type `application/opi-eps+jwt`. An unsigned bundle, which MUST NOT be delivered to a third party (Section 11), is identified by `application/opi-eps+json`. Registration of these media types with IANA is pending.

## 10. Examples (non-normative)

Bundle-level `iat` and `exp` are NumericDate values (seconds since the Unix epoch) per [RFC 7519] Section 2. Element-level `updated` is preference data rather than a JWT claim and remains an RFC 3339 string.

### 10.1 Minimal bundle

```json
{
  "schema": "https://openpreference.org/eps/0.1",
  "iss": "https://example.com",
  "sub": "acct:alice@example.com",
  "iat": 1776349800,
  "exp": 1776353400,
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
  "iss": "https://example.com",
  "sub": "acct:alice@example.com",
  "iat": 1776349800,
  "exp": 1776353400,
  "preferences": [
    {
      "type": "https://openpreference.org/eps/0.1/types/locale",
      "version": "1.0.0",
      "value": "en-CA"
    },
    {
      "type": "https://openpreference.org/eps/0.1/types/dietary/restrictions",
      "version": "1.0.0",
      "value": ["vegetarian"],
      "updated": "2026-03-02T09:15:00Z"
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
- [RFC 7518] JSON Web Algorithms (JWA)
- [RFC 7519] JSON Web Token (JWT)
- [RFC 7565] The 'acct' URI Scheme
- [RFC 8037] CFRG Elliptic Curve Diffie-Hellman (ECDH) and Signatures in JSON Object Signing and Encryption (JOSE)
- [RFC 8174] Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words
- [RFC 8725] JSON Web Token Best Current Practices
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
- [UCUM] The Unified Code for Units of Measure, Regenstrief Institute, https://ucum.org/ucum
