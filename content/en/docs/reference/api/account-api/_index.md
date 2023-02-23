---
title: IAM account API
---

IAM provides a RESTful API that can be used to manage users, group membership, clients, etc.

IAM implements the following endpoints:
* ```/iam/account/{id}/attributes```, providing access to user attributes
* ```/iam/account/{id}/authorities```, providing access to user authorities/roles
* ```/iam/account/me/clients```, providing access to clients owned by the user
* ```/iam/account/find/{option}```, searching users by username/label/e-mail/group/certificate subject
* ```/iam/account/{id}/groups/{groupId}```, providing access to user groups
* ```/iam/account/{id}/managed-groups```, providing access to groups to which a user is manager
* ```/iam/group/{groupId}/group-managers```, listing managers of a certain group
* ```/iam/account/{id}/labels```, providing access to user account labels
* ```/iam/account/{id}/endTime```, managing user membership end time
* ```/iam/account/me/proxycert```, managing user proxy certificate
* ```/iam/account/search```, listing user accounts.

Authentication is required in all the endpoints and the access is based on IAM roles.
Remember that there are three roles in Indigo IAM: Amdin, User, Group Manager.

## User attributes

### GET `/iam/account/{id}/attributes`

Retrieves user attributes. The {id} refers to the account identifier.

Requires that the user has `ROLE_ADMIN`, `ROLE_GM`, or is the one represented by the {id}.

```bash
$ curl -s -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/attributes | jq
[
  {
    "name": "Nickname",
    "value": "Test"
  }
]
```

### PUT `/iam/account/{id}/attributes`

Adds attributes to the user account. It can be done directly through the IAM dashboard by clicking on "Set attribute" button in *Attributes* section of the user homepage.

Requires `ROLE_ADMIN`.

```bash
$ curl -X PUT -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${AT}" -d @attribute.json \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/attributes
```

where ```attribute.json``` is:

```json
{
   "name":"Nickname",
   "value":"Test"
}
```

### DELETE `/iam/account/{id}/attributes`

Deletes user attribute by adding the query parameter at the end of the endpoint.

Requires `ROLE_ADMIN`.

```bash
$ curl -X DELETE -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/attributes?name=Nickname
```

## User roles

### GET `/iam/account/{id}/authorities`

Retrieves user roles.

Requires `ROLE_ADMIN` or `ROLE_GM`.

```bash
$ curl -s -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/authorities | jq
{
  "authorities": [
    "ROLE_GM:c617d586-54e6-411d-8e38-649677980001",
    "ROLE_USER"
  ]
}
```

The above example shows that the Test User is also a group manager (of group with ```c617d586-54e6-411d-8e38-649677980001``` id).

### POST `/iam/account/{id}/authorities`

Adds an authority to the user.

Requires `ROLE_ADMIN`.

```bash
$ curl -X POST -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Bearer ${AT}" -d "authority=ROLE_ADMIN" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/authorities
```

### DELETE `/iam/account/{id}/authorities`

Revokes user authority by specifying the query parameter.

Requires `ROLE_ADMIN`.

```bash
$ curl -X DELETE -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/authorities?authority=ROLE_ADMIN
```

## Managed clients

### GET `/iam/account/me/clients`

Retrieves information about clients owned by the currently authenticated user.

```bash
$ curl -s -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/me/clients | jq
{
  "totalResults": 1,
  "itemsPerPage": 1,
  "startIndex": 1,
  "Resources": [
    {
      "client_id": "client",
      "client_name": "Test Client",
      "redirect_uris": [
        "https://iam.local.io/iam-test-client/openid_connect_login",
        "http://localhost:9090/iam-test-client/openid_connect_login"
      ],
      "scope": "address phone openid profile offline_access read-tasks attr write-tasks email read:/ write:/",
      "created_at": 1658764288643
    }
  ]
}
```

## Account filtering

### GET `/iam/account/find/{option}`

Filters user information by label, e-mail, username, certificate subject or group/notingroup.

Requires `ROLE_ADMIN`.

| Option | Attribute | Value |
| -------- | -------- | -------- |
|   byusername   |   username   | string     |
|   bylabel   |   name   | string     |
|   byemail   |   email   | string     |
|   bycertsubject   |   certificateSubject   | URL-encoded     |

Examples of the available options:
* byusername
    ```bash
    $ curl -s -H "Authorization: Bearer ${AT}" \
      http://localhost:8080/iam/account/find/byusername?username=test | jq
    {
      "totalResults": 1,
      "itemsPerPage": 10,
      "startIndex": 1,
      "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:ListResponse"
      ],
      "Resources": [
        {
          "id": "80e5fb8d-b7c8-451a-89ba-346ae278a66f",
          "meta": {
            "created": "2022-07-26T13:48:35.442+02:00",
            "lastModified": "2022-07-26T13:48:35.442+02:00",
            "location": "http://localhost:8080/scim/Users/80e5fb8d-b7c8-451a-89ba-346ae278a66f",
            "resourceType": "User"
          ...
      ]
    }
    ```

* bylabel
    ```bash
    $ curl -s -H "Authorization: Bearer ${AT}" \
      http://localhost:8080/iam/account/find/bylabel?name=test
    ```

* byemail
    ```bash
    $ curl -s -H "Authorization: Bearer ${AT}" \
      http://localhost:8080/iam/account/find/byemail?email=test.user@gmail.com
    ```

* bycertsubject
    ```bash
    $ curl -s -H "Authorization: Bearer $AT" \
      http://localhost:8080/iam/account/find/bycertsubject?certificateSubject=CN%3dTest%20User%20test%40infn.it%2cO%3dIstituto%20Nazionale%20di%20Fisica%20Nucleare%2cC%3dIT%2cDC%3dtcs%2cDC%3dterena%2cDC=org
    ```

* bygroup/{groupId}
    ```bash
    $ curl -s -H "Authorization: Bearer ${AT}" \
      http://localhost:8080/iam/account/find/bygroup/6a384bcd-d4b3-4b7f-a2fe-7d897ada0dd1
    ```

* notingroup/{groupId}
    ```bash
    $ curl -H "Authorization: Bearer ${AT}" \
      http://localhost:8080/iam/account/find/notingroup/6a384bcd-d4b3-4b7f-a2fe-7d897ada0dd1
    ```

## Group members

### POST `/iam/account/{id}/groups/{groupId}`

Adds user to a group. It can be done directly through the IAM dashboard as explained [here][group membership section].

Requires `ROLE_ADMIN` or `ROLE_GM`.

```bash
$ curl -X POST -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/groups/c617d586-54e6-411d-8e38-649677980004
```


### DELETE `/iam/account/{id}/groups/{groupId}`

Removes user from a specific group.

Requires `ROLE_ADMIN` or `ROLE_GM`.

```bash
$ curl -X DELETE -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/groups/c617d586-54e6-411d-8e38-649677980004
```

## Group managers

### GET `/iam/account/{id}/managed-groups`

Lists the user's managed and not managed groups.

Requires that the user has `ROLE_ADMIN` or is the one represented by the {id}.

```bash
$ curl -s -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/managed-groups | jq
{
  "id": "80e5fb8d-b7c8-451a-89ba-346ae278a66f",
  "username": "test",
  "managedGroups": [
    {
      "name": "Test-001",
      "id": "c617d586-54e6-411d-8e38-649677980001",
      "description": null,
      "parent": null
    }
  ],
  "unmanagedGroups": [
    ...
  ]
}
```

### POST `/iam/account/{id}/managed-groups/{groupId}`

Gives a user represented by {id} `ROLE_GM` privileges of the group identified by {groupId}.

Requires `ROLE_ADMIN`.

```bash
$ curl -X POST -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/managed-groups/c617d586-54e6-411d-8e38-649677980004
```


### DELETE `/iam/account/{id}/managed-groups/{groupId}`

Removes a group manager from a certain group.

Requires `ROLE_ADMIN`.

```bash
$ curl -X DELETE -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/managed-groups/c617d586-54e6-411d-8e38-649677980004
```

### GET `/iam/group/{groupId}/group-managers`

Shows the information of managers in a certain group.

Requires `ROLE_ADMIN` or `ROLE_GM` privileges of the group identified by {groupId}.

```bash
$ curl -s -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/group/c617d586-54e6-411d-8e38-649677980004/group-managers
[
  {
    "id": "80e5fb8d-b7c8-451a-89ba-346ae278a66f",
    "meta": {
      "created": "2022-07-26T13:48:35.442+02:00",
      "lastModified": "2022-07-26T18:19:03.786+02:00",
      "location": "http://localhost:8080/scim/Users/80e5fb8d-b7c8-451a-89ba-346ae278a66f",
      "resourceType": "User"
    },
    "schemas": [
      "urn:ietf:params:scim:schemas:core:2.0:User",
      "urn:indigo-dc:scim:schemas:IndigoUser"
    ],
    "userName": "test",
    "name": {
      "familyName": "User",
      "formatted": "Test User",
      "givenName": "Test"
    },
    ...
  }
]
```

## Account labels

### GET `/iam/account/{id}/labels`

Shows the user account labels.

Requires that the user has `ROLE_ADMIN`, `ROLE_GM`, or is the one represented by the {id}.

```bash
$ curl -s -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/labels
[
  {
    "prefix": "hr.cern",
    "name": "ignore"
  }
]
```

### PUT `/iam/account/{id}/labels`

Adds labels to user account.

Requires `ROLE_ADMIN`.

```bash
$ curl -X PUT -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${AT}" -d @labels.json \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/labels
```

where `labels.json` is:

```json
{
  "prefix": "hr.cern",
  "name": "ignore"
}
```

### DELETE `/iam/account/{id}/labels`

Deletes an account label by adding the query parameter.

Requires `ROLE_ADMIN`.

```bash
$ curl -X DELETE -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/labels?name=ignore
```

## User account expiration time

### PUT `/iam/account/{id}/endTime`

Adds/changes the membership end time of the user. It can be done directly through the IAM dashboard 
by clicking on "Change membership end time" button in the user homepage.

Requires `ROLE_ADMIN`.

```bash
$ curl -X PUT -d @endTime.json \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/80e5fb8d-b7c8-451a-89ba-346ae278a66f/endTime
```

where ```endTime.json``` is:

```json
{
   "endTime":"2022-08-06T02:00:00.000+02:00"
}
```

## Proxy certificate

### PUT `/iam/account/me/proxycert`

Adds user proxy certificate. It can be done directly through the IAM dashboard 
by clicking on "Add managed proxy certificate" button 
appearing on the user homepage after uploading the X.509 certificate.

```bash
$ curl -i -X PUT -d @proxy.json \
  -H "Authorization: Bearer ${AT}" \
  -H "Content-Type: application" \
  http://localhost:8080/iam/account/me/proxycert
```

where `proxy.json` includes only the *certificate_chain* key:

```json
{"certificate_chain":"-----BEGIN CERTIFICATE-----\nMIIFGT...FlCU=\n-----END CERTIFICATE-----"}
```

## List accounts and groups

### GET `/iam/account/search`

Shows the list of IAM accounts.

Requires `ROLE_ADMIN`, `ROLE_GM` or `scim:read` scope.

```bash
$ curl -s -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/account/search
{
  "totalResults": 254,
  "itemsPerPage": 10,
  "startIndex": 1,
  "Resources": [
    {
      "id": "73f16d93-2441-4a50-88ff-85360d78c6b5",
      "meta": {
        "created": "2022-09-21T17:20:14.034+02:00",
        "lastModified": "2022-09-21T17:20:14.034+02:00",
        "location": "http://localhost:8080/scim/Users/73f16d93-2441-4a50-88ff-85360d78c6b5",
        "resourceType": "User"
      },
      "userName": "admin",
      "name": {
        "familyName": "User",
        "formatted": "Admin User",
        "givenName": "Admin"
      },
      "displayName": "admin",
      "active": true,
      "emails": [
        {
          "type": "work",
          "value": "1_admin@iam.test",
          "primary": true
        }
      ]
    },
    ...
```

### GET `/iam/group/search`

Shows the list of IAM groups.

Access granted to all IAM users or `scim:read` scope.

```bash
$ curl -s -H "Authorization: Bearer ${AT}" \
  http://localhost:8080/iam/group/search
{
  "totalResults": 22,
  "itemsPerPage": 10,
  "startIndex": 1,
  "Resources": [
    {
      "id": "6a384bcd-d4b3-4b7f-a2fe-7d897ada0dd1",
      "meta": {
        "created": "2022-09-21T17:20:15.478+02:00",
        "lastModified": "2022-09-21T17:20:15.478+02:00",
        "location": "http://localhost:8080/scim/Groups/6a384bcd-d4b3-4b7f-a2fe-7d897ada0dd1",
        "resourceType": "Group"
      },
      "displayName": "Analysis",
      "urn:indigo-dc:scim:schemas:IndigoGroup": {
        "parentGroup": null,
        "description": "The analysis group",
        "labels": null
      }
    },
    ...
```

[group membership section]: {{< ref "/docs/tasks/administration/group-management/#managing-membership-for-a-group" >}}