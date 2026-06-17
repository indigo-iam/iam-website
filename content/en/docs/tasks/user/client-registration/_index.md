---
title: Registering a client
weight: 2
---

IAM exposes the OpenID Connect/OAuth dynamic client registration functionality
on its own dashboard (it has been migrated by [MitreID][mitreid] dashboard).

In OAuth terminology, a client is an application or service that can interact
with an authorization server for authentication/authorization purposes.

A new client can be registered in the IAM in two ways:

- using the [dynamic client registration API][reg-api];
- via the IAM dashboard (which simply acts as a client to the API mentioned
  above).

## Registering a client using the dashboard

Log into the service and click on the _My Clients_ link on the left
navigation bar:

![dashboard](../images/my_client-dashboard.png)

From the _My client_ link, select _New client_:

![client reg](../images/new-client-reg-0.png)

A form will open that enable you to configure your client:

![client reg](../images/new-client-reg-1.png)

The minimum information you have to provide is:

- _Client name_: choose a name for your client
- _Redirect URI(s)_: one or more redirect URIs for your client; these are
  required if you choose to enable the authorization code flow;

{{% alert title="Info" color="info" %}}
Redirect URIs MUST satisfy the following requirements:

- The URI scheme MUST be either `http` or `https`, except for `edu.kit.data.oidc-agent:/redirect`, which is allowed for oidc-agent
- `http` redirect URIs MUST use a loopback address, as defined in [RFC 8252](https://datatracker.ietf.org/doc/html/rfc8252)
- Redirect URIs MUST NOT contain a fragment component
{{% /alert %}}

![client reg](../images/new-client-reg-2.png)

Remember to select the `offline_access` scope from the __Scopes__ tab if you
want to request refresh tokens for the client being created:

![client reg](../images/new-client-reg-3.png)

{{% alert title="Info" color="info" %}}
Restricted scopes can only be assigned to clients by admin users. As for the __admin__ (`iam:admin.read` and `iam:admin.write`) and __SCIM__ (`scim:read` and `scim:write`) scopes, only admin users can delegate clients to use them. _Client credentials_ clients may obtain access tokens with these privileged/restricted scopes if and only if they have been previously assigned by the administrator.
{{% /alert %}}

You can then click the __Save__ button at the bottom of the page:

![client reg](../images/new-client-reg-4.png)

IAM will then generate client credentials for your client and other information
that will be displayed as follows:

![client reg](../images/new-client-reg-5.png)

[mitreid]: https://github.com/mitreid-connect/OpenID-Connect-Java-Spring-Server/wiki
[reg-api]: {{< ref "/docs/reference/api/oidc-client-registration/" >}}
