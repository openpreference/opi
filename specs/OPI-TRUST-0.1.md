---
title: "OPI-TRUST 0.1: Trust Registry"
doc_id: OPI-TRUST
version: 0.1
status: draft
editor: Robin Monks
issued: 2026-04-16
tags: [open-preferences, opi, trust, draft]
---

# OPI-TRUST 0.1: Trust Registry

## Abstract

This document defines the structure and distribution of trust lists used by preference servers to evaluate requestors. A trust list is a signed collection of verification records attesting to facts about identified requestors, including domain control, legal entity status, industry category, and compliance attestations.

## Status of This Document

This is a draft at version 0.1. It is open for comment and subject to substantive change. No conformance claims against this document are authoritative until a version 1.0 release.

## 1. Introduction

Preference servers operating at the client or user tier defined in [OPI-PD] require reliable information about requestors to apply access policies. OPI-TRUST specifies a signed, subscription-based trust list format. Multiple trust registries MAY coexist. Preference servers MAY subscribe to one or more and compose their results under local policy.

## 2. Terminology

**Trust list.** A signed JSON document enumerating verification records for one or more requestors.

**Registry.** An entity that issues and maintains a trust list.

**Verification record.** A statement that a named requestor has been verified as possessing a specific attribute.

**Verification category.** A typed classification of what is being attested, expressed as a URI.

**Subscriber.** A preference server that consumes one or more trust lists.

Terminology defined in [OPI-EPS] and [OPI-PD] applies to this document.

## 3. Conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHOULD, SHOULD NOT, MAY, and OPTIONAL in this document are to be interpreted as described in [RFC 2119] and [RFC 8174] when, and only when, they appear in all capitals.

## 4. Trust List

A trust list is a JSON object with the following members.

| Member | Type | Required | Description |
|--------|------|----------|-------------|
| `schema` | string | yes | Absolute URI identifying the trust list schema version. For this document: `https://openpreference.org/trust/0.1`. |
| `registry` | string | yes | URI identifying the issuing registry. |
| `issued` | string | yes | RFC 3339 timestamp of list generation. |
| `expires` | string | yes | RFC 3339 timestamp after which the list is stale. |
| `entries` | array | yes | Array of verification entries (Section 5). |

Trust lists MUST be wrapped in a JSON Web Signature envelope per [RFC 7515]. The JWS payload is the serialized trust list.

## 5. Verification Entry

A verification entry is a JSON object with the following members.

| Member | Type | Required | Description |
|--------|------|----------|-------------|
| `entity` | string | yes | URI identifying the verified requestor. Typically an HTTPS URL identifying the requestor's primary domain. |
| `client_id` | string | no | OAuth 2.0 `client_id` associated with the entity, where applicable. |
| `verifications` | array | yes | Array of verification records (Section 6). |
| `revoked` | boolean | no | If true, all prior verifications for this entity are revoked. Default false. |

## 6. Verification Record

A verification record is a JSON object with the following members.

| Member | Type | Required | Description |
|--------|------|----------|-------------|
| `category` | string | yes | URI identifying the verification category (Section 7). |
| `verified_at` | string | yes | RFC 3339 timestamp of verification. |
| `expires` | string | no | RFC 3339 timestamp after which the verification is no longer asserted. |
| `evidence` | string | no | Opaque identifier referencing the underlying evidence, such as a business registration number. |

## 7. Verification Categories

Registries MAY define category URIs under their own namespaces. The following baseline categories are defined by this specification.

### 7.1 Domain control

URI: `https://openpreference.org/trust/0.1/categories/domain-control`

Asserts that the named entity demonstrated control of the DNS domain identified by the entity URI at the time of verification.

### 7.2 Legal entity

URI: `https://openpreference.org/trust/0.1/categories/legal-entity`

Asserts that the named entity is a registered legal entity. The `evidence` member SHOULD carry a jurisdiction-qualified registration identifier.

### 7.3 Industry category

URI prefix: `https://openpreference.org/trust/0.1/categories/industry/`

Asserts that the named entity is classified in a specific industry. The final path segment identifies the industry. Registries SHOULD align with a published taxonomy such as [NAICS] or [ISIC].

### 7.4 Compliance attestation

URI prefix: `https://openpreference.org/trust/0.1/categories/compliance/`

Asserts that the named entity holds a specific compliance attestation. The final path segment identifies the attestation (for example, `.../compliance/hipaa`, `.../compliance/soc2-type2`, `.../compliance/iso-27001`).

## 8. List Subscription

### 8.1 List distribution

A registry MUST publish its current trust list at a stable HTTPS URL. The URL MAY be advertised via WebFinger or distributed out-of-band.

### 8.2 Freshness

Subscribers MUST refresh trust lists before their `expires` timestamp. Subscribers MUST NOT use expired trust lists for access decisions.

### 8.3 Composition

A subscriber MAY consume multiple trust lists. Where categories conflict, the subscriber's local policy determines resolution. Subscribers SHOULD document their resolution policy.

## 9. Signing

Trust lists MUST be signed per [RFC 7515]. Public keys used to verify signatures MUST be distributed as a JWK Set per [RFC 7517] at a stable URL referenced by the registry.

Registries MUST rotate signing keys on a defined schedule and MUST publish key rotation events such that subscribers can validate lists signed under either the previous or current key during a rotation window.

## 10. Revocation

Registries MUST republish trust lists with updated `issued` timestamps whenever entries are added, modified, or revoked. Subscribers MUST apply the most recent list they hold.

Registries MAY publish revocations as delta documents with the same schema, where the `entries` array contains only revoked entities with `revoked` set to true. Delta documents MUST be signed.

## 11. Examples (non-normative)

### 11.1 Trust list

```json
{
  "schema": "https://openpreference.org/trust/0.1",
  "registry": "https://openpreference.org/registry/default",
  "issued": "2026-04-16T00:00:00Z",
  "expires": "2026-04-23T00:00:00Z",
  "entries": [
    {
      "entity": "https://partner-restaurant.example",
      "client_id": "opi-client-abc123",
      "verifications": [
        {
          "category": "https://openpreference.org/trust/0.1/categories/domain-control",
          "verified_at": "2026-04-01T00:00:00Z",
          "expires": "2027-04-01T00:00:00Z"
        },
        {
          "category": "https://openpreference.org/trust/0.1/categories/legal-entity",
          "verified_at": "2026-04-01T00:00:00Z",
          "expires": "2027-04-01T00:00:00Z",
          "evidence": "us:de:6543210"
        },
        {
          "category": "https://openpreference.org/trust/0.1/categories/industry/restaurant",
          "verified_at": "2026-04-01T00:00:00Z",
          "expires": "2027-04-01T00:00:00Z"
        }
      ]
    }
  ]
}
```

### 11.2 Revocation delta

```json
{
  "schema": "https://openpreference.org/trust/0.1",
  "registry": "https://openpreference.org/registry/default",
  "issued": "2026-04-16T12:00:00Z",
  "expires": "2026-04-23T12:00:00Z",
  "entries": [
    {
      "entity": "https://partner-restaurant.example",
      "revoked": true,
      "verifications": []
    }
  ]
}
```

## 12. Security Considerations

Trust list signatures are the sole source of integrity for list content. Compromise of a registry's signing key voids all assertions made under that key.

Subscribers MUST validate the signature, `issued`, and `expires` members of every list before use. Subscribers MUST reject lists whose signing key is not present in the registry's published JWK Set.

## 13. References

### 13.1 Normative

- [OPI-EPS] Open Preference Initiative: Extensible Preference Schema, version 0.1
- [OPI-PD] Open Preference Initiative: Preference Discoverability, version 0.1
- [RFC 2119] Key words for use in RFCs to Indicate Requirement Levels
- [RFC 3339] Date and Time on the Internet: Timestamps
- [RFC 7515] JSON Web Signature (JWS)
- [RFC 7517] JSON Web Key (JWK)
- [RFC 8174] Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words

### 13.2 Informative

- [NAICS] North American Industry Classification System
- [ISIC] International Standard Industrial Classification
