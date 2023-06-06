---
title: Scope Policy API
---

In OpenID Connect and OAuth, [scopes][oauth-scopes] are used to determine the
privileges granted to a client application for a given session. For example,
the `openid` and `profile` scopes are used to obtain access to identity
information about a user, but applications can define their own scopes to
implement domain-specific authorization (e.g., the WLCG `storage.read`  and
`storage.modify` scopes are examples of domain-specific OAuth scopes).

IAM implements two levels of access control on OAuth scopes:

- a client-level vetting, implemented through the [MitreID
  connect][mitreid-connect] library, so that each registered client has a list
  of allowed scopes. When a client requests a scope that is not allowed to get,
  it gets an `invalid_scope` error.

- an identity-related vetting, implemented through the concept of **Scope
  policies**, that can be used to limit access to scopes based on user
  identity.

This behaviour is described in the following picture:

{{< imgproc scope-access-control.png Resize "500x" >}}
{{< /imgproc >}}


## Scope policies

IAM Scope policies provide a mechanism to control access to OAuth
scopes. A scope policy defines:

- a `rule`, which can be `PERMIT` or `DENY`, that determines the behaviour of
  the policy. `PERMIT` policies are used to allow access to scopes, while
  `DENY` policies are used to block access to scopes;

- a scopes selector, i.e. a set of scopes for which the policy applies (and a
  scope `matchingPolicy` used to determine the scope matching algorithm used);

- an `account` or `group` selector, used to determine for which user account or
  group of accounts the policy should apply.

An example scope policy is given below:

```json
{
    "id": 1,
    "description": "Default Permit ALL policy",
    "creationTime": "2019-10-08T13:52:20.000+02:00",
    "lastUpdateTime": "2019-10-08T13:52:20.000+02:00",
    "rule": "PERMIT",
    "matchingPolicy": "EQ",
    "account": null,
    "group": null,
    "scopes": null
  }
```

The policy above is a `PERMIT` policy that allows access to all the scopes to
any account, as clarified by the policy `description`.

In the policy, the `account` and `group` selector are not specified, which
means that the policy would apply to any user account. Also the `scopes`
selector is not specified, which means that the policy would apply to any
scope.

{{% alert color="primary" %}}

The policies that are not explicitly bound to an account or a group are called
**default scope policies**.

{{% /alert %}}


The example policy above is enabled by the default IAM configuration, which in practice
limits the default scope vetting only to the client level.

Here's an another example, now showing two policies:

```json
[{
    "id": 4,
    "description": "Deny access to compute.* scopes to normal users",
    "creationTime": "2019-12-18T15:11:04.000+01:00",
    "lastUpdateTime": "2019-12-18T15:11:04.000+01:00",
    "rule": "DENY",
    "matchingPolicy": "EQ",
    "account": null,
    "group": null,
    "scopes": [
      "compute.create",
      "compute.read",
      "compute.cancel",
      "compute.modify"
    ]
  },
  {
    "id": 13,
    "description": "Allow access to compute.* scopes to wlcg/pilot users",
    "creationTime": "2019-12-18T15:19:20.000+01:00",
    "lastUpdateTime": "2019-12-18T15:19:20.000+01:00",
    "rule": "PERMIT",
    "matchingPolicy": "EQ",
    "account": null,
    "group": {
      "uuid": "25084f30-1d71-4ab2-91e8-11148af16682",
      "name": "wlcg/pilots",
      "location": "https://wlcg.cloud.cnaf.infn.it/scim/Groups/25084f30-1d71-4ab2-91e8-11148af16682"
    },
    "scopes": [
      "compute.create",
      "compute.read",
      "compute.cancel",
      "compute.modify"
    ]
  }]
```

The first policy, as clarified in the description, would block access to the
`compute.create`, `compute.read`, `compute.cancel` and `compute.modify` scopes
to any authenticated user. The second policy would instead allow access to
`compute.*` scopes only to members of the `wlcg/pilot` group (again, as written
in the policy `description`).

The two policies can work together because the IAM policy engine evaluates the
policy in a specific order: account-level policies are applied first, then
group-level policies are applied and finally policies that are not bound to any
specific account or group are applied. This means that a policy defined to
apply at the account or group-level typically wins over a more general one, as
in the example above. Also, scopes being vetted are evaluated only once across
different levels. This means, for example, that a scope that was evaluated at
account-level will not be re-evaluated at group-level: the decision taken for
such scope is not reconsidered.

In more technical terms, the policy engine implements a deny-override policy
composition logic, which means that a deny policy which matches a request at a
given level wins over a permit policy at that same level. For example, if there
are two account-level policies that would apply for scope `s`, one which would
yield a `PERMIT` result, and one that would yield a `DENY`, the `DENY` will
win.

### Scope matching algorithms

IAM currently supports three scope matching algorithms:

- `EQ`: which is the default, uses string equality when comparing requested
  scopes to scopes allowed by the client configuration or by the scope
  policies;

- `REGEXP`: which uses a regular expression evaluation when comparing
  requested scopes to scopes allowed by the client configuration or scope
  policies; 

- `PATH`: which uses a WLCG-specific path-matching logic, described in more
  detail below,  to compare requested scopes to scopes allowed by the client
  configuration or by the scope policies.

`REGEXP` and `PATH` matching algorithms are configured by adding a
`scope.matchers` section to the IAM configuration, as shown in the following
fragment which defines the scope matching algorithm for [WLCG profile
scopes][wlcg-profile]:

```yaml
scope:
  matchers:
    - name: storage.read
      type: path
      prefix: storage.read
      path: /
    - name: storage.create
      type: path
      prefix: storage.create
      path: /
    - name: storage.modify
      type: path
      prefix: storage.modify
      path: /
    - name: wlcg.groups
      type: regexp
      regexp: ^wlcg\.groups(?::((?:\/[a-zA-Z0-9][a-zA-Z0-9_.-]*)+))?$
```

#### PATH scope matching

The `PATH` scope matching algorithm is used to implement a scope-matching logic
compliant with the [WLCG JWT profile rules for storage scope
matching][wlcg-profile], which allow a scope to contain a parametric part
describing a path. Following the [Scitokens model][scitokens], permissions granted on a path
apply transitively to subpaths. To give an example:

```
storage.read:/cms
```

will grant read access to the `/cms` directory and all its content and
subdirectories, but does not grant access to the `/atlas` directory.

Following the same logic, a client allowed to request `storage.read:/example`,
when `PATH` scope matching is configured as described above for the
`storage.read` parametric scope, will be allowed to request the
`storage.read:/example` but also any scope containing subpaths of `/example`,
e.g., `storage.read:/example/subdir/file`.

#### REGEXP scope matching

The `REGEXP` scope matching algorithms uses regular expression matching to
determine whether a scope is allowed for a given client or user.

Taking the parametric `wlcg.groups` scope described above:

```yaml
    - name: wlcg.groups
      type: regexp
      regexp: ^wlcg\.groups(?::((?:\/[a-zA-Z0-9][a-zA-Z0-9_.-]*)+))?$
```

A client allowed to request the scope `wlcg.groups` will also be allowed for
the scope `wlcg.groups:/a/group` or any other group name matching the regexp.


## The Scope policy API

The Scope Policy API is a REST API that allows to manage scope policies.
Access to the API is restricted to administrator users authenticated via web interface
or OAuth clients that have access to the `iam:admin.read` (for read access) or `iam:admin.write` (for write
access) OAuth scopes.

### GET /iam/scope_policies

Returns a JSON representation of the Scope Policies defined for the
organization. 

**Authentication required**: yes

**Authorization required**: `iam:admin.read` scope

**Command example**

```shell
$ curl -i -H "Accept: application/json" -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -X GET https://example.com/iam/scope_policies/
```

#### Success response

**Condition**: Being Authenticated and authorized

**Code**: `200 OK`

**Content**: An array of JSON representations for the scope policies defined in
the organization

```json
[{
  "id": 1,
  "description": null,
  "creationTime": "2018-02-27T07:26:21.000+01:00",
  "lastUpdateTime": "2018-02-27T07:26:21.000+01:00",
  "rule":"PERMIT",
  "matchingPolicy":"EQ",
  "account":null,
  "group":null,
  "scopes":null
}]
```
#### Error responses

**Condition**: User is not authenticated 

**Code**: `401 Unauthorized`

**Content**: 

```json
{
  "error":"unauthorized",
  "error_description":"Full authentication is required to access this resource"
}
```

**OR**

```json
{
  "error":"invalid_token",
  "error_description":"Invalid access token: <Access Token>"
}
```

If an invalid token is provided or no token is provided.

### GET /iam/scope_policies/{id}

Returns the JSON representation for the scope policy identified by `id`.

**Authentication required**: yes

**Authorization required**: `iam:admin.read` scope

**Command example**

```shell
$ curl -i -H "Accept: application/json" -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -X GET https://example.com/iam/scope_policies/1
```

#### Success response

**Condition**: Being Authenticated

**Code**: `200 OK`

**Content**: The JSON representation for the scope policy

```json
{
  "id": 1,
  "description": null,
  "creationTime": "2018-02-27T07:26:21.000+01:00",
  "lastUpdateTime": "2018-02-27T07:26:21.000+01:00",
  "rule":"PERMIT",
  "matchingPolicy":"EQ",
  "account":null,
  "group":null,
  "scopes":null
}
```
#### Error responses

**Condition**: User is not authenticated 

**Code**: `401 Unauthorized`

**Content**: 

```json
{
  "error":"unauthorized",
  "error_description":"Full authentication is required to access this resource"
}
```

**OR**

```json
{
  "error":"invalid_token",
  "error_description":"Invalid access token: <Access Token>"
}
```

If an invalid token is provided or no token is provided.

**Condition**: The Scope Policy {id} is not defined for the organization

**Code**: `404 NOT FOUND`

**Content**: 

```json
{
  "error":"No scope policy found for id: {id}"
}
```

### PUT /iam/scope_policies/{id}

Changes an existing Scope Policy.

**Authentication required**: yes

**Authorization required**: `iam:admin.write` scope

**Data constraints**

Provide a representation of the Scope Policy to be modified.

```json
{
  "id": [positive integer],
  "description": [text, optional, at most 512 chars],
  "rule": [text, allowed values: "PERMIT", "DENY"],
  "matchingPolicy":[text, allowed values: "EQ", "REGEXP", "PATH"],
  "account":[a valid account selector],
  "group":[a valid group selector ],
  "scopes":[a list of scopes, minimum 1 and maximum 255 chars]
}
```

#### Success response

**Condition**: The Scope Policy is modified for the organization

**Code**: `204 No Content`

#### Error responses

**Condition**: Policy validation error 

**Code**: `400 BAD REQUEST`

```json
{
    "error":"Invalid scope policy: rule cannot be empty"
}
```

**Or**

**Condition**: Unauthenticated access 

**Code**: `401 UNAUTHORIZED`

**Content**:

```json
{
    "error":"unauthorized",
    "error_description":"Full authentication is required to access this resource"
}
```
**OR**

```json
{
  "error":"invalid_token",
  "error_description":"Invalid access token: <Access Token>"
}
```

If an invalid token is provided or no token is provided at all

**Or**

**Condition**: Authorization error 

**Code**: `403 FORBIDDEN`

**Content**:

```json
{
    "error":"access_denied",
    "error_description":"Access is denied"
}
```

### POST /iam/scope_policies

Creates a Scope Policy for the organization

**Authentication required**: yes

**Authorization required**: `iam:admin.write` scope

**Data constraints**

Provide a representation of the Scope Policy to be created

```json
{
  "description": [text, optional, at most 512 chars],
  "rule": [text, allowed values: "PERMIT", "DENY"],
  "matchingPolicy":[text, allowed values: "EQ", "REGEXP", "PATH"],
  "account":[a valid account selector],
  "group":[a valid group selector],
  "scopes":[a list of scopes, each scope string should be no longer than 255 chars]
}
```

**Data example** 

```json
{
  "description": "Allow access to compute.* scopes to wlcg/pilot users",
  "rule": "PERMIT",
  "scopes": ["compute.read", "compute.modify", "compute.create", "compute.cancel"],
  "matchingPolicy": "EQ",
  "group": {
       "uuid": "25084f30-1d71-4ab2-91e8-11148af16682"
  }
}
```

#### Success response

**Condition**: The Scope Policy is created for the organization

**Code**: `201 Created`

#### Error responses

**Condition**: Policy validation error 

**Code**: `400 BAD REQUEST`

```json
{
    "error":"Invalid scope policy: rule cannot be empty"
}
```

**Or**

**Condition**: Unauthenticated access 

**Code**: `401 UNAUTHORIZED`

**Content**:

```json
{
    "error":"unauthorized",
    "error_description":"Full authentication is required to access this resource"
}
```

**OR**

```json
{
  "error":"invalid_token",
  "error_description":"Invalid access token: <Access Token>"
}
```

If an invalid token is provided or no token is provided at all

**Or**

**Condition**: Authorization error 

**Code**: `403 FORBIDDEN`

**Content**:

```json
{
    "error":"access_denied",
    "error_description":"Access is denied"
}
```

### DELETE /iam/scope_policies/{id}

Deletes the Scope Policy for the organization.

**Authentication required**: yes

**Authorization required**: `iam:admin.write` scope

**Command example**

```shell
$ curl -i -H "Accept: application/json" -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -X DELETE https://example.com/iam/scope_policies/{id}
```
Where {id} is the Scope Policy id number


#### Success response

**Condition**: The Scope Policy is defined for the organization

**Code**: `204 NO CONTENT`

#### Error responses

**Condition**: The Scope Policy {id} is not defined or inexistent for the organization

**Code**: `404 NOT FOUND`

**Content**: 
```json
{
    "error":"No scope policy found for id: 1"
}
```

**Or**

**Condition**: Unauthenticated access 

**Code**: `401 UNAUTHORIZED`

**Content**:

```json
{
    "error": "unauthorized",
    "error_description": "Full authentication is required to access this resource"
}
```

**OR**

```json
{
  "error":"invalid_token",
  "error_description":"Invalid access token: <Access Token>"
}
```

If an invalid token is provided or no token is provided.

**Or**

**Condition**: Authorization error 

**Code**: `403 FORBIDDEN`

**Content**:

```json
{
    "error": "Access is denied"
}
```

[mitreid-connect]: https://github.com/mitreid-connect/
[oauth-scopes]: https://datatracker.ietf.org/doc/html/rfc6749#section-3.3
[wlcg-profile]: https://github.com/WLCG-AuthZ-WG/common-jwt-profile/blob/master/profile.md#capability-based-authorization-scope
[scitokens]: https://scitokens.org/technical_docs/Claims
