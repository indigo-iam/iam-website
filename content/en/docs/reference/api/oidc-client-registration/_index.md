---
title: OpenID Connect client registration API
---

Starting from version 1.8.0, IAM supports the [OpenID Connect Dynamic Client Registration
API][oidc-dynclientreg] at the `/iam/api/client-registration` endpoint.

{{% alert title="Warning" color="warning" %}}

Support to the dynamic client registration through the [MitreID OpenID Connect server implementation](https://github.com/mitreid-connect/OpenID-Connect-Java-Spring-Server/wiki)
at the `/register` endpoint is deprecated.

{{% /alert %}}

Client registration does not require authentication.

## Example: registering and deleting a client from the command line

With the following template JSON you can easily register a client in the IAM:

```json
{
  "redirect_uris": [
    "https://another.client.example/oidc"
  ],
  "client_name": "another-example-client",
  "contacts": [
    "test@iam.test"
  ],
  "token_endpoint_auth_method": "client_secret_basic",
  "scope": "address phone openid email profile offline_access",
  "grant_types": [
    "refresh_token",
    "authorization_code"
  ],
  "response_types": [
    "code"
  ]
}
```

Save the above JSON into a file (after changing the relevant fields, like
contact, rediret_uris, etc...). We will use the `client-req.json` file name for
this example. You can then use [httpie][httpie] as  follows to generate the
client on the IAM

```
http https://iam.local.io/iam/api/client-registration < client-req.json > client.json
```

If the command terminates correctly, you will have the client configuration
saved in the `client.json` file. You can use [jq][jq] to display in a pretty
way its contents:

```
$ jq . client.json
{
  "client_id": "9bef3e68-f329-4dbf-9c3e-45647c08dfd5",
  "client_secret": "ALMjVauX1tXtU2lZytbxfW7JrEx6-ZhZml8bZcEocYOf1enNdIQaWB0aDUmr-nilGjjrznzbWNNWdh8bP0neyUI",
  "client_name": "another-example-client",
  "redirect_uris": [
    "https://another.client.example/oidc"
  ],
  "contacts": [
    "test@iam.test"
  ],
  "grant_types": [
    "authorization_code",
    "refresh_token"
  ],
  "response_types": [
    "code"
  ],
  "token_endpoint_auth_method": "client_secret_basic",
  "scope": "address phone openid profile offline_access email",
  "reuse_refresh_token": true,
  "dynamically_registered": true,
  "clear_access_tokens_on_refresh": true,
  "require_auth_time": false,
  "registration_access_token": "eyJraWQiOiJyc2ExIiwiYWxnIjoiUlMyNTYifQ.eyJpc3MiOiJodHRwOlwvXC9sb2NhbGhvc3Q6ODA4MFwvIiwiYXVkIjoiOWJlZjNlNjgtZjMyOS00ZGJmLTljM2UtNDU2NDdjMDhkZmQ1IiwiaWF0IjoxNjQ5MzM4OTcxLCJqdGkiOiJmNTFlZWIzMS1kY2RkLTQ5MDctYThlZC1jZTViMGU3ODRiODAifQ.Ec5PCjaaIveIpMPhTcYvJfsnJc9Ag_n46ICcaDhh8GDXupKAARv_fx4oCQPgomSKAz4j1bVRgzrmQsswR8lKmWQZv5fG2BMaRdh9epENArUbTsaPqdUfoMsqo-rqeN-eOcl4I1NChnfCPdBi2rSg2M5y4o9DSqhtQUUnig3-n77X8QhN8ES2i2MFgLkwBkc88nvPFdtSpNHvKjIlaXx6YRA3F7kqv7LHNP5OKfnfXNAFPhKweRRTXamEu_oN3-u0TQbxkg8YdNo4CN8hfMYMKsaCigND765zxFeRXSgNkpfZdV3d-8K5aU6TQoIOR41_kylbOh6rt6TGD4IIosEn5w",
  "registration_client_uri": "https://iam.local.io/iam/api/client-registration/9bef3e68-f329-4dbf-9c3e-45647c08dfd5",
  "created_at": 1649338971207
}
```

and also to extract information from it conveniently from the command line.
For example, you can extract the `client_id` and `registration_access_token`
with the following commands:

```
$ export CID=$(jq -r .client_id client.json)
$ export RAT=$(jq -r .registration_access_token client.json)
```

and use them to delete the client:

```
$ http DELETE https://iam.local.io/iam/api/client-registration/${CID} Authorization:"Bearer ${RAT}"
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: *
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Connection: keep-alive
Content-Language: en-US
Date: Thu, 07 Apr 2022 13:51:40 GMT
Expires: 0
Pragma: no-cache
Server: nginx/1.13.1
Strict-Transport-Security: max-age=31536000 ; includeSubDomains
X-Application-Context: INDIGO IAM:mysql,google,registration,saml,mysql-test:8080
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
```



[oidc-dynclientreg]: https://openid.net/specs/openid-connect-registration-1_0.html
[jq]: https://stedolan.github.io/jq/
[httpie]: https://httpie.org/
