---
title: "JWT profiles support"
linkTitle: "JWT profiles support"
weight: 1
---

Starting with version 1.6.0, IAM introduces support for JWT _profiles_.

A JWT profile is a named set of rules that defines which information is
included in access tokens, id tokens,  userinfo and introspection responses
issued by IAM in an OAuth/OIDC message exchanges.

IAM allows to define a default profile that is used for all clients that are
not explicitly bound to one. This allows to use the same profile across
all clients in an organization.

It's also possible to define the profile to be used at client level, using
scopes, i.e. adding among the scope allowed for the client the name of the
profile the client should use. The scope, in this context, does not identify a
set of permissions but an encoding of authentication/authorization informations
in tokens and responses issued by IAM.

This mechanism enables integration of the same IAM instance with resources
relying on different profiles.

IAM currently supports four JWT profiles:

- the `iam` profile
- the `wlcg` WLCG profile 
- the `aarc` profile
- the `kc` profile


## Setting the default profile

The IAM JWT default profile is set using the `IAM_JWT_DEFAULT_PROFILE`
environment variable, e.g.:

```
IAM_JWT_DEFAULT_PROFILE=iam
```

The default profile will be used for all clients that do not explicitly and
correctly request the use of a profile using scopes. 

## Using scopes to select the JWT profile for a client

To select the profile used by a client, include one of the following scopes
in the list of scopes authorized for a client:

- `iam`, for the IAM profile
- `wlcg`, for the WLCG profile 
- `aarc`, for the AARC profile
- `kc`, for the Keycloak profile

Clients should only link to one profile. When multiple profiles are linked to a
client, IAM falls back to the default profile configured for the IAM instance.


## Using system scopes to control profile selection

MitreID **System Scopes** can be used to control access to profile selection.

The scopes `iam`, `wlcg` and `aarc` used to select the JWT profile are not available in IAM
by default; an administrator has to add all of them manually, as described in the
in [System Scopes documentation][system-scopes].

## Supported JWT profiles

### The IAM profile

This is the default profile for IAM.

With this profile:

- groups are encoded in the `groups` claim; all the groups the user is member
  of are included in the groups claim;

- authentication information (username, email, groups) is not by default
  included in access tokens; this behaviour can be changed by setting the
  `IAM_ACCESS_TOKEN_INCLUDE_AUTHN_INFO=true` environment variable;

- scope information is not by default included in access tokens; this behaviour
  can be changed by setting the `IAM_ACCESS_TOKEN_INCLUDE_SCOPE=true`
  environment variable;

- the `nbf` (not before) claim is not set in access tokens; this behaviour
  can be changed by setting the `IAM_ACCESS_TOKEN_INCLUDE_NBF=true`
  environment variable.

This profile is assigned to clients using the `iam` scope.

### The WLCG profile

This profile implements the [Common JWT Token profile][wlcg-profile].

In particular:

- the `scope` claim is always included in access tokens;
- groups are not included by default in access and ID tokens.

This profile is assigned to clients using the `wlcg` scope.

With this profile:

- groups are encoded in the `wlcg.groups` claim; all the non-optional groups the user is member
  of are included in the wlcg.groups claim;

- authentication information (name, preferred username and email) is not by default
  included in access tokens; this behaviour can be changed by setting the
  `IAM_ACCESS_TOKEN_INCLUDE_AUTHN_INFO=true` environment variable and by requesting
  * `email` scope to include the `email` claim in access tokens;
  * `profile` scope to include the `name` and `preferred_username` claims in access tokens;

- the `nbf` (not before) claim is not set in access tokens; this behaviour
  can be changed by setting the `IAM_ACCESS_TOKEN_INCLUDE_NBF=true`
  environment variable. If the `nbf` claim is included, you can configure how long before 
  the token's issue time it becomes valid by setting the environment variable
  `IAM_ACCESS_TOKEN_NBF_OFFSET_SECONDS=60`. By default, this value is
  60 seconds, meaning the token will be valid starting 60 seconds before it is issued.


#### Requesting groups with the WLCG profile

When the WLCG profile is used, groups are not automatically included in access
tokens and ID tokens, but can be requested using the `wlcg.groups` scope
(which has to be added among the [System Scopes][system-scopes]),
following the rules described in the [WLCG
profile](https://github.com/WLCG-AuthZ-WG/CommonJWTProfile/blob/master/profile.md#scope-based-group-selection).

#### Optional groups

Optional groups are groups whose membership is only asserted in tokens on
explicit request coming from a user.

In order to configure a IAM group as an optional group,
add the `wlcg.optional-group` label to the group.


### The AARC profile

This profile encodes group membership information following the rules defined
by the [AARC G002][aarc-g002] profile document.

In particular:

- groups and organisation name are not included by default in access and ID tokens;
- organisation name can be requested using the `eduperson_scoped_affiliation` scope and it's encoded in the `eduperson_scoped_affiliation` claim;
- groups can be requested using the `eduperson_entitlement` scope and they're encoded as URN in the `eduperson_entitlement` claim.

All the mapping rules are described in the [White Paper for implementation mappings between SAML 2.0 and OpenID Connect in Research and Education](https://docs.google.com/document/d/1b-Mlet3Lq7qKLEf1BnHJ4nL1fq-vMe7fzpXyrq2wp08/edit).

Moreover, the unique identifier `voPersonID` claim is available through this profile and follows the rules defined by the [AARC G026][aarc-g026] and the [voPerson v2.0][voPerson-v2.0] schema version.

`voPersonID` is the community unique identifier and it holds a scoped version of the value accountID given by INDIGO IAM.
This claim is currently present within the following places (given the AARC profile is enabled):

- ID Token
- Access Token
- Userinfo Response
- Introspection Response

An example of an Access Token (produced via the Test client) is the following: 

```json
{
  "sub": "73f16d93-2441-4a50-88ff-85360d78c6b5",
  "voperson_id": "73f16d93-2441-4a50-88ff-85360d78c6b5@indigo-dc",
  "iss": "http://localhost:8080/",
  "eduperson_assurance": [
    "https://refeds.org/assurance",
    "https://refeds.org/assurance/IAP/low"
  ],
  "exp": 1769090046,
  "iat": 1769086446,
  "jti": "64e746a2-71f1-4402-acc9-cf4631005590",
  "client_id": "client"
}
```

From the example, one can see that the value from the `voperson_id` claim is a scoped version of `sub`.<br>
The scope is defined by the issuing authority, in this case `indigo-dc` being an identifier of the issuing system. 

This profile is assigned to clients using the `aarc` scope.

[system-scopes]: {{< ref "docs/reference/configuration/system-scopes" >}}
[wlcg-profile]: https://zenodo.org/record/3460258
[aarc-g002]: https://aarc-project.eu/guidelines/aarc-g002/
[aarc-g026]: https://zenodo.org/record/5504407/files/AARC-G026%20-%20Guidelines%20for%20expressing%20community%20user%20identifiers.pdf
[voPerson-v2.0]: https://github.com/voperson/voperson/blob/draft-2.0.0/voPerson.md#vopersonid-attribute-definition

### The Keycloak profile

This profile allows the integration with a Keycloak environment.

With this profile:

* the `scope` claim is always included in access tokens;
* groups are always included in access and ID tokens; they are encoded in the `roles` claim.

Having **roles** claim instead of **groups** allows integration with the OIDC client - a conteinerised MISP deployment - which requires them to map IAM users.

This profile is assigned to clients using the `kc` scope.