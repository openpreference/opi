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
| `acct:` URI per [RFC 7565] | `acct:alice@example.com` | Preferences of an individual account. |
| `https:` URI at domain root | `https://example.com` | Preferences of the organization represented by the domain. |

The userpart of an `acct:` URI MUST be non-empty, as required by [RFC 7565]. Servers SHOULD treat an `https:` subject with and without a trailing slash as the same subject.

Servers MAY accept additional URI schemes. Servers MUST respond with HTTP 404 for any `resource` value they do not recognize, subject to Section 10.

## 5. Discovery

### 5.1 Query

Requestors discover a subject's preference resource by issuing an HTTPS GET request per [RFC 7033]:

```http
GET /.well-known/webfinger?resource={uri}&rel=https://openpreference.org/rel/preferences
Host: {host}
```

The `host` is the host part of an `acct:` URI, or the host of an `https:` URI.

### 5.2 Discovery document

The response body MUST be a JSON Resource Descriptor per [RFC 7033], served with the `application/jrd+json` media type. The document MUST contain the link relations in Section 5.3.

### 5.3 Link relations

| Link relation | Required | `type` | Description |
|---------------|----------|--------|-------------|
| `https://openpreference.org/rel/preferences` | yes | `application/opi-eps+jwt` | The subject's preference resource (Section 7). |
| `https://openpreference.org/rel/keys` | yes | `application/jwk-set+json` | JWK Set per [RFC 7517] used to verify preference bundle signatures. |
| `https://openpreference.org/rel/auth` | conditional | `application/json` | OAuth 2.0 authorization server metadata per [RFC 8414]. REQUIRED if the preferences link supports the `client` or `user` tier. |

The preferences link MUST carry the following property. Property values are strings, as required by [RFC 7033] Section 4.4.4.5.

| Property URI | Value | Required | Description |
|--------------|-------|----------|-------------|
| `https://openpreference.org/pd/0.1/tiers` | string | yes | Space-separated list of access tiers supported at this `href`. Permitted tokens: `public`, `client`, `user`. Order is not significant. |

## 6. Access Tiers

### 6.1 Public tier

The public tier serves preferences the subject has marked available to any requestor. Retrieval requires no authentication. Responses MUST be signed per [OPI-EPS] Section 9.1.

### 6.2 Client tier

The client tier serves preferences the subject has authorized for release to requestors whose identity is verified. Retrieval requires the requestor to present a credential obtained via OAuth 2.0 Client Credentials grant per [RFC 6749]. Preference servers MAY further restrict access based on verified attributes from a trust registry as specified in [OPI-TRUST].

A preference server MUST record a purpose statement for each client at registration time, in the same form as the `purpose` member in Section 8.1. The registered purpose applies to every client-tier request the client makes and MUST be available to the subject through the audit interface described in Section 13.

### 6.3 User tier

The user tier serves preferences that require explicit user authorization, either per request or under a standing policy. Retrieval requires an access token issued by the authorization server advertised at the `https://openpreference.org/rel/auth` link, obtained through one of the two profiles defined in Section 8. Preference servers MUST support at least one profile and MUST advertise which profiles they support per Section 8.4.

## 7. Retrieval

### 7.1 Request

Requestors retrieve preferences by issuing an HTTPS GET request to the `href` returned in the discovery document:

```http
GET {href}?types={type_list}
Authorization: {auth_scheme} {credential}
```

The `types` query parameter, if present, is a space-separated list of preference type URIs, percent-encoded per [RFC 3986]. A server receiving no `types` parameter MUST return all preferences the requestor is entitled to receive at its authenticated tier (Section 7.4).

### 7.2 Response

The response body MUST be a signed bundle per [OPI-EPS] Section 9.1. The `Content-Type` header MUST be `application/opi-eps+jwt`.

### 7.3 Type filtering

The server MUST return only preferences whose `type` is both listed in the `types` parameter, if present, and within the requestor's entitlement at its authenticated tier (Section 7.4). The server MUST NOT return preferences outside the requested types.

### 7.4 Entitlement

The set of preference types a requestor is entitled to receive is determined by tier.

| Tier | Entitlement |
|------|-------------|
| Public | Every preference the subject has marked available to any requestor. |
| Client | The intersection of the preferences the subject has authorized for release to verified requestors and the OAuth 2.0 `scope` values granted to the client's access token. Each `scope` value is one preference type URI. |
| User | The `preference_types` member of the `authorization_details` bound to the access token (Section 8.1). |

The `types` query parameter narrows the response within the entitlement and MUST NOT expand it. A request whose `types` include a type outside the requestor's entitlement MUST be answered with the entitled subset rather than an error, so that the response does not reveal which types exist.

## 8. User-Tier Authorization

### 8.1 Authorization details

Both profiles convey the request using an `authorization_details` object per [RFC 9396]. Each object MUST include the following members.

| Member | Type | Required | Description |
|--------|------|----------|-------------|
| `type` | string | yes | Fixed value `https://openpreference.org/pd/0.1/authorization_details`. |
| `preference_types` | array of strings | yes | Requested preference type URIs. |
| `purpose` | string | yes | Human-readable purpose string displayed to the user during consent. |
| `retention` | string | yes | Duration per [ISO 8601] indicating the maximum period the requestor will retain the preferences. |

The authorization server MUST present `purpose` and `retention` to the user whenever it obtains consent interactively, and MUST evaluate them against the user's standing policy when consent is not interactive. Access tokens issued under either profile MUST be bound to the granted `authorization_details`, and preference servers MUST enforce `preference_types` as the upper bound on what the token can retrieve (Section 7.3).

### 8.2 RAR profile

The requestor obtains an access token using the authorization code grant per [RFC 6749], including the `authorization_details` parameter in the authorization request per [RFC 9396] Section 3. Public clients MUST use PKCE per [RFC 7636].

### 8.3 UMA profile

The preference server acts as a UMA resource server and the authorization server acts as a UMA authorization server per [UMA 2.0] and [UMA FedAuthz].

1. The preference server MUST register each subject's preference resource with the authorization server, using the preference type URIs it can serve as the `resource_scopes`.
2. A user-tier request that lacks a sufficient token MUST receive HTTP 401 with a `WWW-Authenticate: UMA` header carrying `as_uri` and a permission ticket, per [UMA 2.0] Section 3.2.1.
3. The requestor MUST request a requesting party token at the token endpoint with `grant_type` set to `urn:ietf:params:oauth:grant-type:uma-ticket`, the `ticket`, and the `authorization_details` parameter per [RFC 9396] Section 6 carrying the object in Section 8.1. The `preference_types` MUST be a subset of the scopes named in the permission ticket.
4. The authorization server MUST obtain the resource owner's consent through claims gathering or standing policy as described in Section 8.1, and MUST record `purpose` and `retention` with the granted permission.

### 8.4 Metadata

The authorization server metadata document referenced from the `https://openpreference.org/rel/auth` link MUST include `authorization_details_types_supported` containing `https://openpreference.org/pd/0.1/authorization_details`, per [RFC 9396] Section 10. A server supporting the UMA profile MUST publish the UMA discovery members defined in [UMA FedAuthz] in the same document, which extends [RFC 8414].

## 9. Rate Limiting

### 9.1 Self-hosted endpoints

Self-hosted preference servers SHOULD impose per-source-address rate limits on unauthenticated requests. A default of 60 requests per minute per source address is RECOMMENDED.

### 9.2 Central index endpoints

Preference servers that serve discovery or preference data for subjects outside their primary administrative control MUST require client registration for all requests, including public-tier requests. Per-client rate limits apply.

## 10. Enumeration Resistance

Preference servers MUST NOT distinguish between "subject exists but has no preferences available to this requestor" and "subject does not exist." Both cases MUST return HTTP 404 with a response body of constant size and a response timing of constant order.

## 11. Examples (non-normative)

### 11.1 Discovery request

```http
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
      "type": "application/opi-eps+jwt",
      "properties": {
        "https://openpreference.org/pd/0.1/tiers": "public client user"
      }
    },
    {
      "rel": "https://openpreference.org/rel/keys",
      "href": "https://example.com/.well-known/opi-keys.json",
      "type": "application/jwk-set+json"
    },
    {
      "rel": "https://openpreference.org/rel/auth",
      "href": "https://example.com/.well-known/oauth-authorization-server",
      "type": "application/json"
    }
  ]
}
```

### 11.3 Public-tier retrieval

```http
GET /opi/preferences/alice?types=https%3A%2F%2Fopenpreference.org%2Feps%2F0.1%2Ftypes%2Flocale
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

Preference servers MUST log all non-public-tier requests with sufficient detail to support user-facing audit, including requestor identity, requested types, purpose (the registered purpose for client-tier requests, or the purpose member for user-tier requests), and timestamp. Servers SHOULD provide a user interface for reviewing, modifying, and revoking standing authorizations. Logs MUST be retained in accordance with applicable law.

## 14. References

### 14.1 Normative

- [OPI-EPS] Open Preference Initiative: Extensible Preference Schema, version 0.1
- [OPI-TRUST] Open Preference Initiative: Trust Registry, version 0.1
- [ISO 8601] Date and time, Representations for information interchange
- [RFC 2119] Key words for use in RFCs to Indicate Requirement Levels
- [RFC 3986] Uniform Resource Identifier (URI): Generic Syntax
- [RFC 6749] The OAuth 2.0 Authorization Framework
- [RFC 7033] WebFinger
- [RFC 7515] JSON Web Signature (JWS)
- [RFC 7517] JSON Web Key (JWK)
- [RFC 7565] The 'acct' URI Scheme
- [RFC 7636] Proof Key for Code Exchange by OAuth Public Clients
- [RFC 8174] Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words
- [RFC 8414] OAuth 2.0 Authorization Server Metadata
- [RFC 9396] OAuth 2.0 Rich Authorization Requests
- [UMA 2.0] User-Managed Access 2.0 Grant for OAuth 2.0 Authorization, Kantara Initiative
- [UMA FedAuthz] User-Managed Access 2.0 Federated Authorization for UMA 2.0, Kantara Initiative
