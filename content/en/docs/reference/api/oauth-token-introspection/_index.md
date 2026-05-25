---
title: "OAuth token introspection API"
linkTitle: "OAuth token introspection API"
---

IAM supports the [OAuth token introspection][oauth-token-introspection] specification,
at the `/introspect` endpoint.

The `/introspect` endpoint __requires__ client authentication (e.g. Basic authentication).<br>
In INDIGO IAM, the introspection response includes all the fields listed in the specification, also
the OPTIONAL once.

A typical call to the introspection endpoint is the following:

```bash
$ curl -u $CLIENT_ID:$CLIENT_SECRET https://iam.test.example/introspect -d token=$AT -s | jq.
{
  "active": true,
  "sub": "73f16d93-2441-4a50-88ff-85360d78c6b5",
  "iss": "https://iam.test.example/",
  "token_type": "ACCESS_TOKEN",
  "client_id": "client",
  "aud": [
    "http://example1.com",
    "http://example2.com",
    "http://example3.com"
  ],
  "nbf": 1779705640,
  "scope": "openid email profile offline_access",
  "exp": 1779709300,
  "iat": 1779705700,
  "jti": "afff71ea-a755-4abd-b02b-ba77f6ee9ee0",
  "username": "test"
}
```

## Proxied token introspection

INDIGO IAM also supports for the Proxied token introspection at the `/introspection` endpoint,
as described in [AARC-G052][proxied-token-introspection].

In order to enable the Proxied token introspection one has to configure IAM as per
[documentation](/docs/reference/configuration/proxied-token-introspection).

[oauth-token-introspection]: https://tools.ietf.org/html/rfc7662
[proxied-token-introspection]: https://zenodo.org/records/10205863

