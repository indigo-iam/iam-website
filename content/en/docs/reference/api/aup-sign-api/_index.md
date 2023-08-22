---
title: "AUP signature API"
linkTitle: "AUP signature API"
weight: 1
---

IAM provides a RESTful API that can be used to manage the IAM Acceptable Usage Policy (AUP) signature time.

### GET /iam/aup/signature/{accountId}

Returns a JSON representation of the Acceptable Usage Policy (AUP) for the
organization, some account info (uuid, username, name) and the signature time.

```bash
curl -X GET -H "Authorization: Bearer $AT" \
http://localhost:8080/iam/aup/signature/{accountId}
```

**Authentication required**: yes

**Authorization required**: ROLE\_ADMIN/ROLE\_GM or `iam:admin.read` scope or user with that _accountId_

#### Success response

**Condition**: The AUP is defined for the organization and the user signed it

**Code**: `200 OK`

**Content**: A JSON representation of the AUP, some account info and the AUP signature time

```json
{
   "aup":{
      "url": "http://somehost.example.org/aup",
      "text": "This is my AUP text",
      "description": null,
      "signatureValidityInDays": 0,
      "creationTime": "2022-03-14T17:47:48.000+01:00",
      "lastUpdateTime": "2022-03-14T17:48:03.000+01:00"
   },
   "account":{
      "uuid": "8b7b42fd-0e42-43c5-8254-729aa8f6a12d",
      "username": "test",
      "name": "Test User"
   },
   "signatureTime":"2023-04-27T11:35:20.000+02:00"
}
```

The `/iam/aup/signature` endpoint gives the same result and can be accessed by any authenticated user.

#### Error response

**Condition**: The user did not sign the AUP

**Code**: `404 NOT FOUND`

**Content**: 

```json
{
  "error":"AUP signature not found for user 'test'"
}
```

**Or**

**Condition**: Authorization error

**Code**: `403 FORBIDDEN`

**Content**: 

```json
{
   "error":"insufficient_scope",
   "error_description":"Insufficient scope for this resource",
   "scope":"iam:admin.read"
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

### PATCH /iam/aup/signature/{accountId}

Creates or updates the AUP signature time for the account.

```bash
curl -X PATCH -H "Authorization: Bearer $AT" \
-H "Content-Type: application/json" -d '{"signatureTime":"2023-08-22T12:28:01.627+02:00"}' \
http://localhost:8080/iam/aup/signature/{accountId}
```

**Authentication required**: yes

**Authorization required**: `iam:admin.write` scope

**Data constraints**

Provide a signature time in UTC format 

```json
{
  "signatureTime": "2023-04-27T11:35:20.000+02:00"
}
```

#### Success response

**Condition**: The AUP is defined for the organization

**Code**: `200 OK`

**Content**: A JSON representation of the AUP, some account info and the AUP signature time

```json
{
   "aup":{
      "url": "http://somehost.example.org/aup",
      "text": "This is my AUP text",
      "description": null,
      "signatureValidityInDays": 0,
      "creationTime": "2022-03-14T17:47:48.000+01:00",
      "lastUpdateTime": "2022-03-14T17:48:03.000+01:00"
   },
   "account":{
      "uuid": "8b7b42fd-0e42-43c5-8254-729aa8f6a12d",
      "username": "test",
      "name": "Test User"
   },
   "signatureTime":"2023-04-27T11:35:20.000+02:00"
}
```

#### Error response

**Condition**: Authorization error

**Code**: `403 FORBIDDEN`

**Content**: 

```json
{
   "error":"insufficient_scope",
   "error_description":"Insufficient scope for this resource",
   "scope":"iam:admin.write"
}
```

**Or**

**Condition**:  Missing or invalid AUP signature time

**Code**: `400 BAD REQUEST`
