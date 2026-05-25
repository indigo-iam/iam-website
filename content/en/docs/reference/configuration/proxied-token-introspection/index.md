---
title: "Proxied token introspection"
linkTitle: "Proxied token introspection"
weight: 6
---

INDIGO IAM implements the Proxied token introspection specification, described in [AARC-G052][proxied-token-introspection].
Basically, it allows to return an introspection response also for tokens issued by another, trusted
Authorization Server (AS) -- by default IAM can only introspect its own issued tokens.

## Trust

IAM trusting another Authorization Server means that the IAM operator has registered an OAuth Client
into the external AS, and that Client is allowed to make requests to the introspection endpoint of the external
AS via Basic authentication. The Basic authentication requires a `client_id` and `client_secret`, obtained during
the Client registration into the external AT and saved into IAM configuration.

## Configuration

The configuration for the Proxied token introspection -- which is disabled by default --
follows the `oidc` [Spring profiles](/docs/reference/configuration/#spring-profiles) structure.
The `oidc` profile was originally introduced to support [login with external OIDC Providers](/docs/reference/configuration/external-authentication/oidc), but has been extended since IAM v1.14.0.
So, to enable the Proxied token
introspection, you should override the `application-oidc.yml` file
as described in [Overriding default configuration](/docs/reference/configuration/#overriding-default-configuration-templates), with something like:

```yaml
oidc:
  providers:
    # Name of the external AS trusted by the current IAM instance
  - name: iam-remote
    # Enable the Proxied token introspection for this AS.
    # It is disabled by default
    allow-proxied-introspection: true
    # External AS URL, appearing as 'iss' claim in the token
    issuer: https://iam-remote.test.example/
    client:
      # ClientID configured in the trusted AS
      clientId: <client-id>
      # Client secret configured in the trusted AS
      clientSecret: <client-secret>
    loginButton:
      # Do not allow login with this external provider through
      # a login button. Allowed by default
      visible: false
```

To recap, we may encounter here three use-cases:
- `oidc.providers[].allow-proxied-introspection = true && oidc.providers[].loginButton.visible = false`:
  allows remote proxied introspection but not login
- `oidc.providers[].allow-proxied-introspection = false && oidc.providers[].loginButton.visible = true`:
  default behavior, only allows login with external OIDC providers
- `oidc.providers[].allow-proxied-introspection = true && oidc.providers[].loginButton.visible = true`:
  allows login with external OIDC providers and supports for Proxied token introspection.

### Cache

The well-known endpoint response of external OIDC providers is cached when doing Proxied token introspection.

Here is a snippet on how to configure this specific cache (when the property `oidc.providers[].allow-proxied-introspection` is set to `true`):

```yaml
cache:
  # Enable the caching mechanism in IAM.
  # When set to 'false', no-one kind of cache will be used.
  # The default behavior is an in-memory cache
  enabled: true
  # Refresh period for the external OIDC providers well-known
  # endpoint cache (in seconds)
  oidc-discovery-cleanup-period-secs: 86400
  # Allow to cache the information into a Redis service
  redis:
    enabled: false
```

## Test

In order to test if the Proxied token introspection is working, please obtain an access token from the external
AS, save it into the `AT` variable and cross-check that it is issued by the expected Authorization Server --
one of those listed in the `oidc.providers[].issuer` property:

```bash
$ echo $AT | cut -d . -f2 | base64 -d 2>/dev/null | jq .iss
"https://iam-remote.test.example/"
```

Let's make an introspection response to another IAM instance, e.g. https://iam.test.example/ where the Proxied token introspection is enabled and the instance is configured such to trust https://iam-remote.test.example/.
We should obtain something like:

```bash
$ curl https://iam.test.example/introspect -u $CLIENT_ID:$CLIENT_SECRET$ -d token=$AT -s | jq .
{
  "active": true,
  "sub": "73f16d93-2441-4a50-88ff-85360d78c6b5",
  "iss": "https://iam-remote.test.example/",
  "token_type": "ACCESS_TOKEN",
  "client_id": "<client-id>",
  "aud": [
    "http://rs.example.com"
  ],
  "nbf": 1769082266,
  "scope": "openid email profile",
  "auth_time": 1769082293,
  "exp": 1769085925,
  "iat": 1769082326,
  "jti": "afff71ea-a755-4abd-b02b-ba77f6ee9ee0",
  "username": "test"
}
```

So, if we check the logs of https://iam.test.example/ we will see that the introspection response is forwarded
to https://iam-remote.test.example/:

```
iam-be  | 2026-05-25 17:42:36.872  INFO 135 --- [nio-8080-exec-1] .i.m.i.c.o.d.DefaultOidcDiscoveryService : Discovering OIDC configuration for issuer: https://iam-remote.test.example/
iam-be  | 2026-05-25 17:42:36.999  INFO 135 --- [nio-8080-exec-1] .i.m.i.c.o.d.DefaultOidcDiscoveryService : Discover response status code: 200 OK
iam-be  | 2026-05-25 17:42:37.035  INFO 135 --- [nio-8080-exec-1] i.i.m.i.c.o.i.IamIntrospectionService    : Introspection result for token issued by https://iam-remote.test.example/: active=true
```

### With docker compose

An example of the Proxied token introspection where all the necessary services are deployed via docker compose
is available [here](https://github.com/indigo-iam/iam/tree/develop/compose/proxied-introspection).

[proxied-token-introspection]: https://zenodo.org/records/10205863
