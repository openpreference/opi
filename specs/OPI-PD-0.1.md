---
title: "OPI-PD 0.1: Preference Discoverability"
doc_id: OPI-PD
version: 0.1
status: draft
editor: Robin Monks
issued: 2026-04-16
tags: [open-preferences, opi, pd, draft]
---

# OPI-PD 0.1: Preference Discoverability

## Abstract

This document defines how preference data described by [OPI-EPS] is discovered and retrieved. It specifies a WebFinger-based discovery mechanism, a three-tier access model, and authorization protocols for scoped retrieval. Requestor verification is specified in [OPI-TRUST].

## Status of This Document

This is a draft at version 0.1. It is open for comment and subject to substantive change. No conformance claims against this document are authoritative until a version 1.0 release.

## 1. Introduction

OPI-PD specifies how a service locates a subject's preference resource and retrieves a scoped preference bundle under user-controlled access rules. Discovery uses WebFinger per [RFC 7033]. Retrieval uses HTTP under one of three authentication models: unauthenticated (public tier), client-authenticated (client tier), or user-authorized (user tier).

## 2. Terminology

**Requestor.** An entity that seeks to retrieve a preference bundle for a subject.

**Preference server.** An HTTP endpoint that serves preference bundles for one or more subjects.

**Discovery document.** A JSON Resource Descriptor (JRD) returned by WebFinger containing pointers to a subject's preference resources.

**Access tier.** One of three defined levels of access: public, client, or user.

**Trust registry.** A signed list of verified requestors, as specified in [OPI-TRUST].

Terminology defined in [OPI-EPS] applies to this document.

## 3. Conformance

The key words MUST, MUST NOT, REQUIRED, SHALL, SHOULD, SHOULD NOT, MAY, and OPTIONAL in this document are to be interpreted as described in [RFC 2119] and [RFC 8174] when, and only when, they appear in all capitals.

## 4. Subject Identity

A subject is identified by a URI passed as the WebFinger `resource` parameter. Conforming preference servers MUST accept the following forms.

| Form | Example | Semantics |
|------|---------|-----------|
| `acct:` with local part | `acct:alice@example.com` | Preferences of an individual account. |
| `acct:` without local part | `acct:@example.com` | Preferences of the organization represented by the domain. |
| `https:` URI at domain root | `https://example.com` | Equivalent to `acct:@example.com`. |

Servers MAY accept additional URI schemes. Servers MUST respond with HTTP 404 for any `resource` value they do not recognize, subject to Section 10.

## 5. Discovery

### 5.1 Query

Requestors discover a subject's preference resource by issuing an HTTPS GET request per [RFC 7033]:

```
GET /.well-known/webfinger?resource={uri}&rel=https://openpreference.org/rel/preferences
Host: {host}
```

The `host` is the authority component of the `resource` URI.

### 5.2 Discovery document

The response body MUST be a JSON Resource Descriptor per [RFC 7033]. The document MUST contain at least one `links` entry whose `rel` member is `https://openpreference.org/rel/preferences`.

### 5.3 Link properties

Each preference link in the JRD MUST include the following `properties`.

| Property URI | Value | Description |
|--------------|-------|-------------|
| `https://openpreference.org/pd/0.1/tiers` | JSON-encoded array of strings | Access tiers supported at this `href`. Permitted values: `public`, `client`, `user`. |
| `https://openpreference.org/pd/0.1/auth` | string | URL of the OAuth 2.0 authorization server metadata document per [RFC 8414]. REQUIRED if `client` or `user` tier is supported. |
| `https://openpreference.org/pd/0.1/jwks` | string | URL of the JWK Set per [RFC 7517] used to sign preference bundles. |

## 6. Access Tiers

### 6.1 Public tier

The public tier serves preferences the subject has marked available to any requestor. Retrieval requires no authentication. Responses MUST be signed per [OPI-EPS] Section 9.1.

### 6.2 Client tier

The client tier serves preferences the subject has authorized for release to requestors whose identity is verified. Retrieval requires the requestor to present a credential obtained via OAuth 2.0 Client Credentials grant per [RFC 6749]. Preference servers MAY further restrict access based on verified attributes from a trust registry as specified in [OPI-TRUST].

### 6.3 User tier

The user tier serves preferences that require explicit user authorization, either per request or under a standing policy. Retrieval requires the requestor to present an access token obtained via OAuth 2.0 Rich Authorization Requests per [RFC 9396] or User-Managed Access 2.0 per [UMA 2.0].

## 7. Retrieval

### 7.1 Request

Requestors retrieve preferences by issuing an HTTPS GET request to the `href` returned in the discovery document:

```
GET {href}?scopes={scope_list}
Authorization: {auth_scheme} {credential}
```

The `scopes` query parameter, if present, is a comma-separated list of preference type URIs. A server receiving no `scopes` parameter MUST return all preferences the requestor is entitled to receive at its authenticated tier.

### 7.2 Response

The response body MUST be a JWS-wrapped EPS bundle per [OPI-EPS]. The `Content-Type` header MUST be `application/opi-eps+jwt`.

### 7.3 Scope filtering

The server MUST return only preferences matching the requested scopes that the requestor is entitled to receive at its authenticated tier. The server MUST NOT return preferences outside the requested scopes.

## 8. Rich Authorization Requests

User-tier access MUST use Rich Authorization Requests per [RFC 9396]. Each `authorization_details` object MUST include the following members.

| Member | Type | Required | Description |
|--------|------|----------|-------------|
| `type` | string | yes | Fixed value `https://openpreference.org/pd/0.1/authorization_details`. |
| `preference_types` | array of strings | yes | Requested preference type URIs. |
| `purpose` | string | yes | Human-readable purpose string displayed to the user during consent. |
| `retention` | string | yes | ISO 8601 duration indicating the maximum period the requestor will retain the preferences. |

## 9. Rate Limiting

### 9.1 Self-hosted endpoints

Self-hosted preference servers SHOULD impose per-source-address rate limits on unauthenticated requests. A default of 60 requests per minute per source address is RECOMMENDED.

### 9.2 Central index endpoints

Preference servers that serve discovery or preference data for subjects outside their primary administrative control MUST require client registration for all requests, including public-tier requests. Per-client rate limits apply.

## 10. Enumeration Resistance

Preference servers MUST NOT distinguish between "subject exists but has no preferences available to this requestor" and "subject does not exist." Both cases MUST return HTTP 404 with a response body of constant size and a response timing of constant order.

## 11. Examples (non-normative)

### 11.1 Discovery request

```
GET /.well-known/webfinger?resource=acct:alice@example.com&rel=https://openpreference.org/rel/preferences
Host: example.com
```

### 11.2 Discovery response

```json
{
  "subject": "acct:alice@example.com",
  "links": [
    {
      "rel": "https://openpreference.org/rel/preferences",
      "href": "https://example.com/opi/preferences/alice",
      "properties": {
        "https://openpreference.org/pd/0.1/tiers": ["public", "client", "user"],
        "https://openpreference.org/pd/0.1/auth": "https://example.com/.well-known/oauth-authorization-server",
        "https://openpreference.org/pd/0.1/jwks": "https://example.com/.well-known/opi-keys.json"
      }
    }
  ]
}
```

### 11.3 Public-tier retrieval

```
GET /opi/preferences/alice?scopes=https://openpreference.org/eps/0.1/types/locale
Host: example.com
```

### 11.4 User-tier Rich Authorization Request

```json
{
  "authorization_details": [
    {
      "type": "https://openpreference.org/pd/0.1/authorization_details",
      "preference_types": [
        "https://openpreference.org/eps/0.1/types/dietary/restrictions"
      ],
      "purpose": "Adapt menu display to dietary restrictions.",
      "retention": "PT1H"
    }
  ]
}
```

## 12. Security Considerations

All requests and responses MUST use TLS. Preference servers MUST validate client credentials and access tokens on every request. Access tokens SHOULD be short-lived. Discovery documents SHOULD be served with HTTP caching headers consistent with the subject's expected rate of change.

## 13. Privacy Considerations

Preference servers MUST log all non-public-tier requests with sufficient detail to support user-facing audit, including requestor identity, requested scopes, stated purpose, and timestamp. Servers SHOULD provide a user interface for reviewing, modifying, and revoking standing authorizations. Logs MUST be retained in accordance with applicable law.

## 14. References

### 14.1 Normative

- [OPI-EPS] Open Preference Initiative: Extensible Preference Schema, version 0.1
- [OPI-TRUST] Open Preference Initiative: Trust Registry, version 0.1
- [RFC 2119] Key words for use in RFCs to Indicate Requirement Levels
- [RFC 6749] The OAuth 2.0 Authorization Framework
- [RFC 7033] WebFinger
- [RFC 7515] JSON Web Signature (JWS)
- [RFC 7517] JSON Web Key (JWK)
- [RFC 8174] Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words
- [RFC 8414] OAuth 2.0 Authorization Server Metadata
- [RFC 9396] OAuth 2.0 Rich Authorization Requests

### 14.2 Informative

- [UMA 2.0] User-Managed Access 2.0 Grant for OAuth 2.0 Authorization, Kantara Initiative
