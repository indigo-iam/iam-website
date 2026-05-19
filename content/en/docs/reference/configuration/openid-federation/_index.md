---
title: "OpenID Federation 1.0"
linkTitle: "OpenID Federation"
weight: 1
---

{{% alert title="Info" color="info" %}}
**OpenID Federation support is experimental**.  
A basic model has been implemented, without metadata policies and trust marks.
{{% /alert %}}

[OpenID Federation 1.0](https://openid.net/specs/openid-federation-1_0.html) defines a mechanism that allows an OpenID Provider (OP, or in OAuth terms, _Authorization Server_) and a Relying Party (RP, or OAuth _Client_) with no pre-existing relationship to trust each other through **Trust Chains**. This enables an OP to accept OAuth/OIDC requests from a Relying Party without requiring prior manual registration.

A **Trust Chain** is established by third-party authorities. The authority at the origin of the chain is the **Trust Anchor**, and any authorities between the Trust Anchor and the target entity (OpenID Provider or Relying Party) are **Intermediate Authorities**. These roles are conceptually similar to root and intermediate Certificate Authorities (CAs) in the Public Key Infrastructure (PKI).

A Relying Party trusts one or more Trust Anchors. If any of these Trust Anchors provides a valid path to the target OP, the Relying Party can establish trust with that OpenID Provider.
Conversely, an OP also trusts one or more Trust Anchors. If any of these Trust Anchors provides a valid path to the target Relying Party, the OpenID Provider can establish trust with that Relying Party.

Technically speaking, a Trust Chain is a sequence of JWTs that are issued by a leaf entity, zero or more Intermediate Authorities, and a Trust Anchor.

## Entity Statement

In OpenID Federation, each participant (OpenID Provider, Relying Party or federation authority) is called *Entity*. Every Entity publishes a signed JSON document, called **Entity Configuration**, at a well-known URL (`/.well-known/openid-federation`). This document contains the Entity’s metadata (for example OIDC metadata, federation parameters) and its public keys for signature verification.

An **Entity Statement** is the signed assertion that an Entity makes about another. It conveys metadata, policies and trust information along the trust chain. In practice, the Entity Configuration is a special type of Entity Statement that an Entity issues about itself (a “self-statement”). Intermediate authorities and Trust Anchors issue Entity Statements about subordinate Entities to build the trust chain.

This mechanism allows RP and OP to automatically discover and validate each other’s metadata and establish trust without manual configuration.

INDIGO IAM can act both as OP and RP (leaf entity), and publishes its Entity Configuration at `/.well-known/openid-federation` endpoint.
An example here:

```json
{
  "sub": "http://iam.example.org/",
  "metadata": {
    "openid_provider": {
      "response_types_supported": [
        "code",
        "token"
      ],
      "client_registration_types_supported": [
        "explicit",
        "automatic"
      ],
      "federation_registration_endpoint": "http://iam.example.org/iam/api/oid-fed/client-registration",
      "request_uri_parameter_supported": false,
      "scopes_supported": [
        "openid",
        "profile",
        "email",
        "address",
        "phone",
        "offline_access",
        "eduperson_scoped_affiliation",
        "eduperson_entitlement",
        "eduperson_assurance",
        "entitlements",
        "aarc",
        "voperson_scoped_affiliation",
        "voperson_id",
        "voperson_external_affiliation",
        "wlcg.groups"
      ],
      "issuer": "http://iam.example.org/",
      "authorization_endpoint": "http://iam.example.org/authorize",
      "userinfo_endpoint": "http://iam.example.org/userinfo",
      "claims_supported": [
        "sub",
        "name",
        "preferred_username",
        "given_name",
        "family_name",
        "middle_name",
        "nickname",
        "profile",
        "picture",
        "zoneinfo",
        "locale",
        "updated_at",
        "email",
        "email_verified",
        "organisation_name",
        "groups",
        "wlcg.groups",
        "external_authn"
      ],
      "jwks_uri": "http://iam.example.org/jwk",
      "subject_types_supported": [
        "public",
        "pairwise"
      ],
      "id_token_signing_alg_values_supported": [
        "HS256",
        "HS384",
        "HS512",
        "RS256",
        "RS384",
        "RS512",
        "ES256",
        "ES384",
        "ES512",
        "PS256",
        "PS384",
        "PS512",
        "none"
      ],
      "registration_endpoint": "http://iam.example.org/iam/api/client-registration",
      "token_endpoint": "http://iam.example.org/token"
    },
    "openid_relying_party": {
      "client_registration_types": [
        "explicit"
      ],
      "application_type": "web",
      "jwks_uri": "http://iam.example.org/jwk",
      "redirect_uris": [
        "http://iam.example.org/openid_connect_login"
      ]
    }
  },
  "jwks": {
    "keys": [
      {
        "kty": "RSA",
        "e": "AQAB",
        "use": "sign",
        "kid": "rsa1",
        "alg": "RS256",
        "n": "4GRvJuFantVV3JdjwQOAkfREnwUFp2znRBTOIJhPamyH4gf4YlI5PQT79415NV4_HrWYzgooH5AK6-7WE-TLLGEAVK5vdk4vv79bG7ukvjvBPxAjEhQn6-Amln88iXtvicEGbh--3CKbQj1jryVU5aWM6jzweaabFSeCILVEd6ZT7ofXaAqan9eLzU5IEtTPy5MfrrOvWw5Q7D2yzMqc5LksmaQSw8XtmhA8gnENnIqjAMmPtRltf93wjtmiamgVENOVPdN-93Nd5w-pnMwEyoO6Q9JqXxV6lD6qBRxI7_5t4_vmVxcbbxcZbSAMoHqA2pbSMJ4Jcw-27Hct9jesLQ"
      }
    ]
  },
  "iss": "http://iam.example.org/",
  "authority_hints": [
    "https://ia.example.org"
  ],
  "exp": 1759591980,
  "iat": 1759505580
}
```

## OpenID Client Registration

OpenID Federation defines two methods for Client registration that both rely on Trust Chains:

* **Explicit Registration** – the Relying Party explicitly registers with the OpenID Provider by submitting its metadata through the OP’s federation registration endpoint. This allows the OP to verify the Client’s trust chain and apply policies before issuing credentials.

* **Automatic Registration** – the OpenID Provider automatically onboards a RP on the basis of its signed Entity Configuration and trust chain, without requiring a separate registration request.

{{% alert title="Info" color="info" %}}
In the current implementation, **Explicit Client Registration is based solely on the RP’s self-signed Entity Configuration**.  
Support for Explicit Client Registration with RP's Trust Chain will be introduced in future versions.
{{% /alert %}}

### Explicit registration

In its OP role, INDIGO IAM exposes the `federation_registration_endpoint`, enabling the explicit registration flow as defined by the OpenID Federation 1.0 specification.  
In this flow, a Relying Party (RP) actively submits its signed Entity Configuration to INDIGO IAM, which validates the request and, if accepted, issues a new Client registration.

The federation registration endpoint is:

```bash
POST /iam/api/oid-fed/client-registration
Content-Type: application/entity-statement+jwt
```

The request body contains the RP’s signed Entity Configuration (i.e. a JWT).

Example (decoded payload):

```json
{
  "iss": "https://rp.example.org",
  "sub": "https://rp.example.org",
  "metadata": {
    "openid_relying_party": {
      "client_name": "Example RP",
      "redirect_uris": ["https://rp.example.org/callback"],
      "grant_types": ["authorization_code"],
      "response_types": ["code"],
      "token_endpoint_auth_method": "client_secret_basic",
      "client_registration_types": ["explicit"]
    }
  },
  "jwks": {
    "keys": [
      {
        "kty": "RSA",
        "kid": "rsa1",
        "n": "....",
        "e": "AQAB"
      }
    ]
  },
  "iat": 1720000000,
  "exp": 1720003600,
  "aud": "https://iam.example.org/",
  "authority_hints": ["https://ta.example.org"]
}
```

If IAM successfully creates a Client registration for the RP, it returns a signed Entity Statement (JWT).
This Entity Statement represents the OP’s signed assertion about the RP and includes the RP’s metadata and trust information.

Example (decoded payload):

```json
{
  "iss": "https://iam.example.org/",
  "sub": "https://rp.example.org",
  "iat": 1720003700,
  "exp": 1720007300,
  "metadata": {
    "openid_relying_party": {
      "client_id": "bae7daf6-515b-44de-96e1-1c2ef44d50c8",
      "client_secret": "XXX",
      "redirect_uris": ["https://rp.example.org/callback"],
      "token_endpoint_auth_method": "client_secret_basic"
    }
  },
  "authority_hints": ["https://ta.example.org"],
  "aud": "https://rp.example.org",
  "trust-anchor": "https://ta.example.org"
}
```

### Automatic registration

Automatic registration is triggered when the RP sends an Authorization Request to IAM (OP)’s Authorization Endpoint.
In this flow, IAM automatically registers the RP if it is able to successfully resolve and validate the RP’s trust chain and if a common Trust Anchor exists between the two parties.

IAM may obtain the RP’s trust information either:

* by resolving the RP’s Entity Configuration starting from its `client_id`/`entity_id`, or
* through a `trust_chain` explicitly provided by the RP.

The Authorization Request is conveyed as a signed Request Object passed through the `request` parameter.
The Request Object MUST contain the claims required by OpenID Federation, including `iss`, `sub`, `aud`, `client_id`, `jti`, and MAY additionally include `iat`, `exp` and `trust_chain`.  
If the trust evaluation succeeds, IAM automatically creates a Client registration for the RP.

Unlike Explicit Client Registration, where the `client_id` is assigned by IAM, in Automatic Client Registration the `client_id` corresponds to the RP’s `entity_id`.
No `client_secret` is issued. The RP is authenticated via proof of possession of a private key corresponding to one of the public keys advertised in its Entity Configuration.

## Configuration

To enable OpenID Federation support, activate the `openid-federation` profile, for example `-Dspring.profiles.active=openid-federation`.

The configuration expects the following properties:

```yml
openid-federation:
  # URL of one or more trusted trust anchors.
  # It may also be a comma-separated list
  trust-anchors: https://ta.example.com
  entity-configuration:
    # Validity duration (in seconds) of the generated
    # Entity Configuration document
    expiration-seconds: 86400
    # One or more federation authorities that this entity trusts.
    # It may be a comma-separated list
    authority-hints: https://ta.example.com,https://ia.example.com
    federation-entity:
      # Human-readable name of the organization operating this
      # federation entity. Filling this property is not mandatory
      organization-name:
      # Contact info of administrators (e.g. email addresses).
      # It may also be comma-separated. Not mandatory
      contacts:
      # URI of a logo representing the entity. Not mandatory
      logo-uri:
```
