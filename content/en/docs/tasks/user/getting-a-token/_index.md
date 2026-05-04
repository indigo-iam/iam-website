---
title: Obtaining an IAM access token from a CLI
weight: 1
---

A token can be obtained from a command-line interface (CLI) in different ways:

- using `oidc-agent`
- using scripts linked to this page, with the resource-owner password
  credentials flow (**deprecated**) or the device code flow
- using `curl`.

In this section we recommend the installations of a set of tools that can help
in managing tokens.

## `oidc-agent`

[oidc-agent][oidc-agent] is a useful tool to easily get and manage access
tokens for command-line applications.

### Installing oidc-agent

See [oidc-agent installation
instructions](https://indigo-dc.github.io/oidc-agent/installation/install).

#### Quick Almalinux9 installation instructions

This recipe shows how to quickly install `oidc-agent` on Almalinux9.

Install the `oidc-agent` repofile

```bash
cd /etc/yum.repos.d
wget https://repo.data.kit.edu//data-kit-edu-almalinux9.repo
```

refresh the cache and install oidc-agent with

```bash
dnf makecache
dnf install oidc-agent
```

### Bootstrapping oidc-agent

The first thing to do is to start `oidc-agent`.
This can be done issuing the following command:

```bash
$ eval $(oidc-agent)
Agent pid 62088
```

### Registering a client

In order to obtain a token out of IAM, a user needs a client registered.
`oidc-agent` can automate this step and store client credentials securely
on the user machine. This is a one-time operation, you do not need a new
client every time you need a token.

A new client can be registered using the `oidc-gen` command, as follows:

```bash
$ oidc-gen -w device wlcg
```

The `-w device` instructs `oidc-agent` to use the device code flow for the
authentication, which is the recommended way with IAM.

oidc-agent will display a list of different providers that can be used for
registration:

```bash
[1] https://wlcg.cloud.cnaf.infn.it/
[2] https://iam-test.indigo-datacloud.eu/
...
[20] https://oidc.scc.kit.edu/auth/realms/kit/
```

Select one of the registered providers, or type a custom issuer (for IAM, the
last character of the issuer string is always a `/`, e.g.
`https://wlcg.cloud.cnaf.infn.it/`).

Then oidc-agent asks for the scopes, typing `max` (without quotes) allows to
get all the allowed scopes.

oidc-agent will register a new client and store the client credentials and a
refresh token locally in encrypted form (the agent will ask for a password from
the user).

As mentioned, `oidc-gen` is meant to be used only once, when you need a client registered.

### Managing oidc-agent account configurations

To see the list of locally configured accounts use the following command:

```bash
> oidc-add -l

The following account configurations are usable:
DEEP-ac
XDC
atlas
...
cms-voms-importer
escape
escape-monitoring
iam-test
infn-cloud
wlcg
```

You can then load an account in the agent again with the oidc-add command, as follows:

```bash
oidc-add wlcg
```

Once you've loaded the account, you can use `oidc-token` to get tokens for that account.

### Getting tokens with `oidc-token`

Tokens can be obtained using the `oidc-token` command, as follows:

```bash
oidc-token wlcg
```

This will request a token with all the scopes requested at client registration
time. To limit the scopes included in the token, the `-s` flag can be used, as
follows:

```bash
oidc-token -s storage.read:/ wlcg
```

The token audience can be limited using the `--aud` flag,

```bash
oidc-token --aud example.audience -s storage.read:/ wlcg
```
### Accessing oidc-agent client configuration

Often is useful to get details about the client configuration generated 
by oidc-agent. This can be done with the `oidc-gen -p` command, that is
used to print a client configuration, as in the following example:

```bash
❯ oidc-gen -p wlcg
Enter decryption password for account config 'wlcg':******
{
	"name":	"wlcg",
	"client_name":	"oidc-agent:wlcg",
	"issuer_url":	"https://wlcg.cloud.cnaf.infn.it/",
	"device_authorization_endpoint":	"https://wlcg.cloud.cnaf.infn.it/devicecode",
	"daeSetByUser":	0,
	"client_id":	"*****",
	"client_secret":	"*****",
	"refresh_token":	"****.*****.",
	"cert_path":	"",
	"scope":	"storage.create:/ openid offline_access storage.read:/ eduperson_scoped_affiliation storage.modify:/ wlcg wlcg.groups eduperson_entitlement",
	"audience":	"",
	"redirect_uris":	["edu.kit.data.oidc-agent:/redirect", "http://localhost:8080", "http://localhost:47947", "http://localhost:4242"],
	"username":	"",
	"password":	"",
	"client_id_issued_at":	1600444894,
	"registration_access_token":	"****",
	"registration_client_uri":	"https://wlcg.cloud.cnaf.infn.it/register/****",
	"token_endpoint_auth_method":	"client_secret_basic",
	"grant_types":	["urn:ietf:params:oauth:grant-type:device_code", "refresh_token"],
	"response_types":	["token"],
	"application_type":	"web",
	"cert_path":	"",
	"audience":	""
}
```

## Scripts

### Obtaining a token with the password flow

{{% alert title="Warning" color="warning" %}}

For security reasons, this grant type is __deprecated__ in OAuth2.1
and it may no longer work in INDIGO IAM.

{{% /alert %}}

The [password flow][oauth-password-flow] allows a user to get a token from the
IAM by using the IAM local credentials (i.e. the username/password credentials
setup at IAM registration time).

In order to use the password flow, a non-privileged user has to:

1. register a client following the instructions given in [the client
  registration][client-registration] section
2. Note down the `client_id` of the generated client and ask an IAM
   administrator to enable the password flow for such client
3. Wait until the client as the password flow enabled
4. Use a script similar to the one given [here][get-token-script] (or write
   your own following the recommendations of the [RFC][oauth-password-flow]) to
   obtain a token out of the IAM


### Obtaining a token with the device code flow

The [device code flow][oauth-device-code-flow] allows a user to get a token
from the IAM from a CLI interface while using an external browser for the
authentication step. This is convenient since:

- it does not require any authorization from administrators (the device code
  flow can be requested by the user at client registration time)
- it allows the user to authenticate with any of the authentication mechanisms
  supported by the IAM
- it __does not__ expose the user credentials to the client application

For nitty and gritty details on how the flow works, see the
[RFC][oauth-device-code-flow].

After having registered a client with the device flow enabled (see [client
registration section][client-registration-ref]), the device code flow can be
used to obtain a token using a script like the one [here][dc-get-token-script]
which does the following:

- contacts the device flow endpoint to start a device flow authentication and
  authorization
- prints code information on the terminal
- waits for user input to proceed and obtain the token(s)

## CURL

The aforementioned OAuth flows may also be directly performed with the `curl` command,
which is exactly the way the scripts obtain a token.

Here we show the `client_credentials` flow, specified in section 4.4 of the
[RFC6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4). It is tought for
service accounts, i.e. for scripts able to secure credentials (NOT public clients)
and which can be obtain a token with capabilities. The client credential flow does not
require a user approval step since it is not bounded to any user nor any user attributes,
such as groups. A typical client credential request could be

```bash
$ curl -u client-cred:secret http://localhost:8080/token -d grant_type=client_credentials -d scope=read-tasks -s | jq
{
  "access_token": "eyJraWQiOiJy...",
  "token_type": "Bearer",
  "expires_in": 3599,
  "scope": "read-tasks"
}
```

### Set custom token expiration

INDIGO IAM supports the `expires_in` request parameter for custom token expiration,
which is applied when asking for an access token at the `/token` endpoint.
The token expiration may only be set __shorter__ than the lifetime configured for the
Client (typically 3600 seconds) -- visible by Administrators. In case a longer
lifetime is requested, IAM will shorten the token lifetime to the configured one.

A typical token request with custom lifetime (here 10 seconds) in case of `client_credentials` flow
is

```bash
$ curl -u client-cred:secret http://localhost:8080/token -d expires_in=10 -d grant_type=client_credentials -d scope=read-tasks -s | jq
{
  "access_token": "eyJraWQiOiJy...",
  "token_type": "Bearer",
  "expires_in": 9,
  "scope": "read-tasks"
}
```

[oauth-password-flow]: https://tools.ietf.org/html/rfc6749#section-4.3
[oauth-device-code-flow]: https://tools.ietf.org/html/draft-ietf-oauth-device-flow-09
[get-token-script]: https://gist.github.com/andreaceccanti/7d863db5ce3f43c74123a2cea8b8f9ff
[dc-get-token-script]: https://gist.github.com/andreaceccanti/5b69323b89ce08321e7b5236de503600
[oidc-agent]: https://indigo-dc.github.io/oidc-agent
[client-registration-ref]: {{< ref "/docs/tasks/user/client-registration/" >}}
