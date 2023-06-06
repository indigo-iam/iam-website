---
title: IAM Token management API 
---

The IAM server has a RESTful API used to list and revoke active access and
refresh tokens.

Access to the API is limited to users with administrator privileges authenticated via a web session or
to access token containing predefined restricted System Scopes.  
All examples assume authorization via OAuth2 bearer token; e.g.

```
GET /iam/api/access-tokens/13 HTTP/1.1
Host: example.com
Authorization: Bearer h480djs93hd8
```

These are the tokens REST API endpoints:

**Accessing tokens**:

- [GET /iam/api/access-tokens](#get-iamapiaccess-tokens)
- [GET /iam/api/access-tokens/:id](#get-iamapiaccess-tokensid)
- [GET /iam/api/refresh-tokens](#get-iamapirefresh-tokens)
- [GET /iam/api/refresh-tokens/:id](#get-iamapirefresh-tokensid)
- [DELETE /iam/api/access-tokens/:id](#delete-iamapiaccess-tokensid)
- [DELETE /iam/api/refresh-tokens/:id](#delete-iamapirefresh-tokensid)

**Deleting tokens**:

- [GET /iam/api/access-tokens](#get-iamapiaccess-tokens)
- [GET /iam/api/access-tokens/:id](#get-iamapiaccess-tokensid)
- [GET /iam/api/refresh-tokens](#get-iamapirefresh-tokens)
- [GET /iam/api/refresh-tokens/:id](#get-iamapirefresh-tokensid)
- [DELETE /iam/api/access-tokens/:id](#delete-iamapiaccess-tokensid)
- [DELETE /iam/api/refresh-tokens/:id](#delete-iamapirefresh-tokensid)


## GET /iam/api/access-tokens

Retrieves the paginated list of all the active access tokens. Returns results in _application/json_.

Requires `iam:admin.read` scope.

Parameters:

| Name | Description |
|:------------:|-------------|
| `count`<br/>**integer**| Specifies the desired maximum number of query results per page. |
| `startIndex`<br/>**integer**| The 1-based index of the first query result. |
| `userId`<br/>**string**| Filters by userName. |
| `clientId`<br/>**string**| Filters by clientId. |

Example:

```
GET http://example.com:8080/iam/api/access-tokens
```

```json
{
  "totalResults": 1,
  "itemsPerPage": 1,
  "startIndex": 1,
  "resources": [
    {
      "id": 6,
      "value": "eyJhbGciOiJSUzI1NiJ9.eyJleHAiOjE0MDEzMTQ1MjksImF1ZCI6WyJhOGFmNzUzYy1mMzI0LTRlNDAtYTE3Ny04N2RmYzA2MjQ5YjciXSwiaXNzIjoiaHR0cDpcL1wvbG9jYWxob3N0OjgwODBcL29wZW5pZC1jb25uZWN0LXNlcnZlci13ZWJhcHBcLyIsImp0aSI6Ijg4ZjM4OGE4LTk1NzctNGQyMC1hZTZjLWMyMDMxOGQ1OWJjNiIsImlhdCI6MTQwMTMxMDkyOX0.HYnNxRvGRdKFykVChL-hdxszcFBvygkeUmc8_iv2Jl4MU-jPJVzMnTwKJbCMWBjeBp8hrb0Dgd9XbnHUDyXxwj8MDrWQEH3QnwYJGRW9JFWjHMGfKDQDFY6Ffl3OFERVbyoB2ObiGTUgbw4Nkl1L1ihuMpMAc5nKi0rk3QXcS1M",
      "scopes": [
        "openid",
        "phone",
        "email",
        "address",
        "profile"
      ],
      "expiration": "2014-05-28T18:02:09-0400",
      "client": {
        "id": 2,
        "clientId": "iam-test-client",
        "contacts": [
          "andrea.ceccanti@cnaf.infn.it"
        ],
        "ref": "https://iam-test.indigo-datacloud.eu/api/clients/2"
      },
      "user": {
        "id": "e1eb758b-b73c-4761-bfff-adc793da409c",
        "userName": "andrea",
        "ref": "https://iam-test.indigo-datacloud.eu/scim/Users/e1eb758b-b73c-4761-bfff-adc793da409c"
      },
      "idToken": {
        "id": 5,
        "ref": "https://iam-test.indigo-datacloud.eu/access-tokens/5"
      }
    }
  ]
}
```

## GET /iam/api/access-tokens/:id

Retrieves all the information about the access token identified by **id** and returns results in _application/json_.

Requires `iam:admin.read` scope.

```
GET http://example.com:8080/iam/api/access-tokens/6
```

```json
{
  "id": 6,
  "value": "eyJhbGciOiJSUzI1NiJ9.eyJleHAiOjE0MDEzMTQ1MjksImF1ZCI6WyJhOGFmNzUzYy1mMzI0LTRlNDAtYTE3Ny04N2RmYzA2MjQ5YjciXSwiaXNzIjoiaHR0cDpcL1wvbG9jYWxob3N0OjgwODBcL29wZW5pZC1jb25uZWN0LXNlcnZlci13ZWJhcHBcLyIsImp0aSI6Ijg4ZjM4OGE4LTk1NzctNGQyMC1hZTZjLWMyMDMxOGQ1OWJjNiIsImlhdCI6MTQwMTMxMDkyOX0.HYnNxRvGRdKFykVChL-hdxszcFBvygkeUmc8_iv2Jl4MU-jPJVzMnTwKJbCMWBjeBp8hrb0Dgd9XbnHUDyXxwj8MDrWQEH3QnwYJGRW9JFWjHMGfKDQDFY6Ffl3OFERVbyoB2ObiGTUgbw4Nkl1L1ihuMpMAc5nKi0rk3QXcS1M",
  "scopes": [
    "openid",
    "phone",
    "email",
    "address",
    "profile"
  ],
  "expiration": "2014-05-28T18:02:09-0400",
  "client": {
    "id": 2,
    "clientId": "iam-test-client",
    "contacts": [
      "andrea.ceccanti@cnaf.infn.it"
    ],
    "ref": "https://iam-test.indigo-datacloud.eu/api/clients/2"
  },
  "user": {
    "id": "e1eb758b-b73c-4761-bfff-adc793da409c",
    "userName": "andrea",
    "ref": "https://iam-test.indigo-datacloud.eu/scim/Users/e1eb758b-b73c-4761-bfff-adc793da409c"
  },
  "idToken": {
    "id": 5,
    "ref": "https://iam-test.indigo-datacloud.eu/access-tokens/5"
  }
}
```

## GET /iam/api/refresh-tokens


Retrieves the paginated list of all the active refresh tokens. Returns results in _application/json_.

Requires `iam:admin.read` scope.

Parameters:

| Name | Description |
|:------------:|-------------|
| `count`<br/>**integer**| Specifies the desired maximum number of query results per page. |
| `startIndex`<br/>**integer**| The 1-based index of the first query result. |
| `userId`<br/>**string**| Filters by userId. |
| `clientId`<br/>**string**| Filters by clientId. |

Example:

```
GET http://example.com:8080/iam/api/refresh-tokens
```

```json
{
  "totalResults": 1,
  "itemsPerPage": 1,
  "startIndex": 1,
  "resources": [
  	 {
	  "id": 1083,
	  "value": "eyJhbGciOiJub25lIn0.eyJqdGkiOiIxMTdmMWRkOS1iOWViLTQ5MjctYThkMS1hYzQ4NjIwYWQzOWYifQ.",
	  "scopes": [
	    "openid",
	    "phone",
	    "email",
	    "address",
	    "profile"
	  ],
	  "expiration": "2014-05-28T18:02:09-0400",
	  "client": {
	    "id": 2,
	    "clientId": "iam-test-client",
	    "contacts": [
	      "andrea.ceccanti@cnaf.infn.it"
	    ],
	    "ref": "https://iam-test.indigo-datacloud.eu/api/clients/2"
	  },
	  "user": {
	    "id": "e1eb758b-b73c-4761-bfff-adc793da409c",
	    "userName": "andrea",
	    "ref": "https://iam-test.indigo-datacloud.eu/scim/Users/e1eb758b-b73c-4761-bfff-adc793da409c"
	  }
	}
  ]
}
```

## GET /iam/api/refresh-tokens/:id

Retrieves all the information about the refresh token identified by **id** and returns results in _application/json_.

Requires `iam:admin.read` scope.

```
GET http://example.com:8080/iam/api/refresh-tokens/1083
```

```json
{
  "id": 1083,
  "value": "eyJhbGciOiJub25lIn0.eyJqdGkiOiIxMTdmMWRkOS1iOWViLTQ5MjctYThkMS1hYzQ4NjIwYWQzOWYifQ.",
  "scopes": [
    "openid",
    "phone",
    "email",
    "address",
    "profile"
  ],
  "expiration": "2014-05-28T18:02:09-0400",
  "client": {
    "id": 2,
    "clientId": "iam-test-client",
    "contacts": [
      "andrea.ceccanti@cnaf.infn.it"
    ],
    "ref": "https://iam-test.indigo-datacloud.eu/api/clients/2"
  },
  "user": {
    "id": "e1eb758b-b73c-4761-bfff-adc793da409c",
    "userName": "andrea",
    "ref": "https://iam-test.indigo-datacloud.eu/scim/Users/e1eb758b-b73c-4761-bfff-adc793da409c"
  }
}
```

## DELETE /iam/api/access-tokens/:id

Revoke the access token identified by **id**.

Requires `iam:admin.write` scope.

```
DELETE http://example.com:8080/iam/api/access-tokens/6
```

```
204  AccessToken revoked
```


## DELETE /iam/api/refresh-tokens/:id

Revoke the access token identified by **id**.

Requires `iam:admin.write` scope.

```
DELETE http://example.com:8080/iam/api/refresh-tokens/1083
```

```
204  RefreshToken revoked
```

More details can be read at the [complete API Reference](tokens_api/index.html).
