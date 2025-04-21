---
title: "Certificate linking requests API"
linkTitle: "Certificate linking requests API"
---


IAM Login Service provides a RESTful API to create and manage certificate linking requests.

## POST /iam/cert_link_requests

Create a new certificate linking request.
The body of the request must contain a JSON with the following parameters:

| Name | Value | Optional or Required | Description |
|:------------:|-------------|-------------|-------------|
| **label**<br/>`string`| A non empty string | Required | Certificate label |
| **pemEncodedCertificate**<br/>`string`| A PEM-encoded string | The PEM string or both the subject and issuer must be present | Certificate content |
| **subjectDn**<br/>`string`| RFC2253-formatted subject DN | The PEM string or both the subject and issuer must be present | Certificate subject |
| **issuerDn**<br/>`string`| One of the known certification authorities, found in the `IAM_X509_TRUST_ANCHORS_DIR` folder | The PEM string or both the subject and issuer must be present | Certificate issuer |
| **notes**<br/>`string`| Any string | Optional, default is `""` | A note to the administrator |

**Response**:
```json
{
    "uuid": "39c647dd-67a7-485f-90dc-82f42df8573f",
    "userUuid": "73f16d93-2441-4a50-88ff-85360d78c6b5",
    "userFullName": "Admin User",
    "username": "admin",
    "label": "mytest",
    "subjectDn": "CN=test123",
    "issuerDn": "CN=Test CA,O=IGI,C=IT",
    "status": "PENDING",
    "creationTime": 1722956330508,
    "lastUpdateTime": 1722956330508
}
```

## GET /iam/cert_link_requests

Returns a paginated list of certificate linking requests.
The list can be filtered by username, certificate subject name or request status.
Users with administrative privileges can list all requests;
other users only the requests they submitted.

Available parameters:

| Name | Value | Optional or Required | Description |
|:------------:|-------------|-------------|-------------|
| **username**<br/>`string`| A valid username | Optional | Username of the account who made the request |
| **subjectDn**<br/>`string`| RFC2253-formatted subject DN | Optional | Subject of the requested certificate |
| **status**<br/>`string`| `PENDING`, `APPROVED`, `REJECTED` | Optional | Status of the request |
| **startIndex**<br/>`integer`| [1-n] | 1 | The 1-based index of the first query result |
| **count**<br/>`integer`| [0-n] | 10 | The number of results to collect |

**Response**:
```json
{
    "totalResults": 1,
    "itemsPerPage": 1,
    "startIndex": 1,
    "Resources": [
        {
            "uuid": "39c647dd-67a7-485f-90dc-82f42df8573f",
            "userUuid": "73f16d93-2441-4a50-88ff-85360d78c6b5",
            "userFullName": "Admin User",
            "username": "admin",
            "label": "mytest",
            "subjectDn": "CN=test123",
            "issuerDn": "CN=Test CA,O=IGI,C=IT",
            "status": "PENDING",
            "creationTime": 1722956330508,
            "lastUpdateTime": 1722956330508
        }
    ]
}
```

## GET /iam/cert_link_requests/{uuid}

Returns the details about a certificate linking request.

**Response**:
```json
{
    "uuid": "39c647dd-67a7-485f-90dc-82f42df8573f",
    "userUuid": "73f16d93-2441-4a50-88ff-85360d78c6b5",
    "userFullName": "Admin User",
    "username": "admin",
    "label": "mytest",
    "subjectDn": "CN=test123",
    "issuerDn": "CN=Test CA,O=IGI,C=IT",
    "status": "PENDING",
    "creationTime": 1722956330508,
    "lastUpdateTime": 1722956330508
}
```

## DELETE /iam/cert_link_requests/{uuid}

Deletes a certificate linking request.
Administrators can delete any request, users can only delete the `PENDING` requests
they previously submitted.

**Response**: `204 Ok`

## POST /iam/cert_link_requests/{uuid}/approve

Approves a certificate linking request.
Only administrators can approve requests.

**Response**:
```json
{
    "uuid": "d3d3dc74-b92f-42be-86ff-0a38a2c6d287",
    "userUuid": "73f16d93-2441-4a50-88ff-85360d78c6b5",
    "userFullName": "Admin User",
    "username": "admin",
    "label": "mytest",
    "subjectDn": "CN=test123",
    "issuerDn": "CN=Test CA,O=IGI,C=IT",
    "status": "APPROVED",
    "creationTime": 1722956648238,
    "lastUpdateTime": 1722956674208
}
```

## POST /iam/cert_link_requests/{uuid}/reject?motivation=

Rejects a certificate linking request.
A `motivation` parameter is required.
Only administrators can reject requests.

**Response**:
```json
{
    "uuid":"d098436e-6b0f-42cc-a21d-9a1246efa551",
    "userUuid":"73f16d93-2441-4a50-88ff-85360d78c6b5",
    "userFullName":"Admin User",
    "username":"admin",
    "label":"mytest",
    "subjectDn":"CN=test1233",
    "issuerDn":"CN=Test CA,O=IGI,C=IT",
    "status":"REJECTED",
    "motivation":"reject motivation",
    "creationTime":1722956985661,
    "lastUpdateTime":1722957048461
}
```