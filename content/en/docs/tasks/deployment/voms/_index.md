---
title: "Deploying the IAM VOMS attribute authority"
weight: 120
description: >
  Instructions on how to deploy and configure the IAM VOMS attribute authority micro-service
---

The IAM [VOMS attribute authority][voms-aa] (AA) provide a backward-compatible
[VOMS][voms] support for a Virtual Organization managed with IAM.

## Deployment architecture

![VOMS AA deployment](../img/iam-voms-aa.png)

The VOMS attribute authority can access the IAM database and encode IAM groups
and other attributes in a standard VOMS attribute certificate. This means in
practice that IAM can act both as an OAuth/OpenID Connect authorization server
and as a VOMS server for a given organization. TLS termination and client VOMS
atttribute certificate parsing and validation is delegated to an [NGINX VOMS module][nginx-httpg-voms],
which can de deployed as a sidecar service that just protects the
VOMS AA or also in front of the IAM backend application. The two scenarios are
depicted above.

In order to deploy a VOMS attribute authority, you are recommended to use the following
docker images:

- `cnafsd/nginx-httpg-voms`, for a NGINX server configured with the [VOMS module][nginx-httpg-voms],
which replaces [OpenResty VOMS][openresty-voms]
- `indigoiam/iam-voms-aa:{{< param voms_aa_version >}}`, for the VOMS AA service

Deployment from packages is not currently supported for the VOMS attribute
authority.

## NGINX VOMS module configuration

The NGINX VOMS module requires:

- the IGTF trust anchors properly configured; see the [EGI trust anchors][egi-trustanchors]
container.
- The certificate and private key to use: the certificate issuer must be one of
the CA present in the EGI trust anchors. See the specific section below.
- the `vomsdir` folder, with the VOMS LSC configuration generated starting from
  the VOMS attribute authority X.509 credential.

To generate the LSC configuration, you need to get the subject and issuer of the
VOMS AA X.509 credential and put them in a file named as the fully qualified
domain name of the VOMS attribute authority with the `.lsc` extension.

Let's assume that the IAM VOMS AA will answer on `voms.test.example` for the VO
`example.vo`.  The following command will generate the appropriate LSC file
(`X509_VOMS_DIR` can be set to any directory where you have writing privileges):

```console
> mkdir -p ${X509_VOMS_DIR}/example.vo
> openssl x509 -in voms_local_io.cert.pem -noout -subject -issuer -nameopt compat | \
  gsed -e 's/^subject=//' -e 's/^issuer=//' > \
  ${X509_VOMS_DIR}/example.vo/voms.test.example.lsc
```

For an example NGINX configuration, see the one use in [VOMS-enabled NGINX container][nginx-httpd-voms-config].

## VOMS AA configuration

The VOMS AA is a Spring Boot application that shares the persistence
layer implementation with IAM, and as such can inspect the IAM database. It can share
the same environment file as the login service it is associated with.

### Using environment variables

The following table shows the environment variables for VOMS AA configuration in common with IAM
login service, related with the access to the database:

| Name | Description | Default |
|------|-------------|---------|
| IAM_DB_HOST | Hostname of database server | localhost |
| IAM_DB_PORT | Port of database server | 3306 |
| IAM_DB_NAME | IAM database name | iam |
| IAM_DB_URL_PARAMS | The custom list of URL connection parameters. It must start with "?" character, if defined. The default value overrides the session time zone setting on the server to "UTC" and disables SSL usage | (*) |
| IAM_DB_USERNAME | Username for accessing the database | iam |
| IAM_DB_PASSWORD | Password for accessing the database | pwd |
| IAM_DB_MAX_ACTIVE | The default maximum database connection pool size | 50 |
| IAM_DB_MIN_IDLE | The minimum number of idle connections the pool attempts to maintain | 8 |
| IAM_DB_VALIDATION_QUERY | A configuration property used to validate the health of a database connection before it is handed out from the pool | SELECT 1 |

(*) ?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true

Moreover, the VOMS AA related environment variables are:

| Name | Description | Default |
|------|-------------|---------|
| VOMS_AA_BINDING_ADDRESS | Address the server binds to (network interface) | 0.0.0.0 |
| VOMS_AA_PORT | Port the server listens on | 8080 |
| VOMS_AA_VONAME | Name of the Virtual Organization (VO) | test |
| VOMS_AA_FORWARD_HEADERS_STRATEGY | Strategy for handling forwarded HTTP headers (e.g. from proxies). Set "native" if you're behind a proxy | none |
| VOMS_AA_TLS_CERTIFICATE_PATH | Path to the TLS server certificate file | /certs/hostcert.pem |
| VOMS_AA_TLS_PRIVATE_KEY_PATH | Path to the TLS private key file | /certs/hostkey.pem |
| VOMS_AA_TLS_TRUST_ANCHORS_DIR | Directory containing trusted CA certificates | /etc/grid-security/certificates |
| VOMS_AA_TLS_TRUST_ANCHORS_REFRESH_INTERVAL_SECS | Interval (in seconds) to reload trusted CA certificates | 14400 |
| VOMS_AA_OPTIONAL_GROUP_LABEL | Attribute name used for optional group membership | wlcg.optional-group |
| VOMS_AA_VOMS_ROLE_LABEL | Label name used for VOMS roles | voms.role |
| VOMS_AA_USE_LEGACY_FQAN_ENCODING | Enable legacy encoding format for FQAN attributes | false |

{{% alert title="Warning" color="warning" %}}
A top level group equal to `VOMS_AA_VO` must be defined in IAM. The attributes appearing in the VOMS proxy include all the sub-groups of the parent group equal to the VO name. Top level groups different from the VO name may still be used for group-based authorization with JWTs, but will not appear in the proxy.
{{% /alert %}}

### Using YAML file

In order to configure VOMS AA through a YAML file, you need to place the file
at `/workspace/config/application.yml` in the container.
The file content is shown below, together with the default value of the properties.

```yaml
server:
  address: 0.0.0.0
  port: 8080
  use-forward-headers: true
  max-http-header-size: 16000

spring:
  main:
    banner-mode: "off"

  jpa:
    open-in-view: false

  flyway:
    enabled: false

  datasource:
    url: jdbc:mysql://db:3306/iam?useLegacyDatetimeCode=false&serverTimezone=UTC&useSSL=false
    username: iam
    password: pwd
    max-active: 50
    minimum-idle: 8
    validation-query: SELECT 1

voms:
  tls:
    certificate-path: /certs/hostcert.pem
    private-key-path: /certs/hostkey.pem
    trust-anchors-dir: /etc/grid-security/certificates
    trust-anchors-refresh-interval-secs: 14400
  aa:
    host: 0.0.0.0
    port: 8080
    vo-name: test
    optional-group-label: wlcg.optional-group
    voms-role-label: voms.role
    use-legacy-fqan-encoding: false
```

Assuming that the configuration file is locally named as `voms-aa.yml`, you
typically need to add the following volume definition to the VOMS AA section of your Podman/Docker
compose file, in order to map it into the proper application file:

```yaml
        volumes:
            - /path/to/voms-aa.yml:/workspace/config/application.yml:ro,Z
```

*Note: `Z` at the end of the line is required only if you use SELinux*

In addition, you need to configure the EGI trust anchors and the VOMS AA certificate and
private key, as described below.

## Configuration of EGI trust anchors and VOMS AA certificate

Both the VOMS-enabled NGINX service and the VOMS AA service require the proper configuration
of the container, the certificate and private key to use and the EGI trust anchors. If using
Podman/Docker Compose, it is typically done with:

```yaml
        volumes:
            - /path/to/certs/voms.test.example.cert.pem:/certs/hostcert.pem:ro,Z
            - /path/to/certs/voms.test.example.cert.key:/certs/hostkey.pem:ro,Z
            - /etc/grid-security/certificates:/etc/grid-security/certificates:ro,Z
```

The line to map EGI trust anchor may be different if you use [EGI trust anchors][egi-trustanchors]
container. See the Podman/Docker Compose example for more information.

Note that the private key must be readable by the user running the service. As it is not a good
practice to make it world readable, it may be necessary to adjust its ownership.

## Podman/Docker Compose example

For an example of the container configurations, see the [VOMS AA Docker compose
file][voms-aa-compose].

[openresty-voms]: https://baltig.infn.it/cnafsd/ngx_http_voms_module
[voms-aa]: https://github.com/indigo-iam/iam/tree/master/iam-voms-aa
[nginx-httpd-voms-config]: https://github.com/indigo-iam/iam/blob/master/compose/voms-deploy/assets/nginx/conf.d/voms.test.example.conf
[voms-aa-compose]: https://github.com/indigo-iam/iam/blob/master/compose/voms-deploy/docker-compose.yml
[voms]: http://italiangrid.github.io/voms/
[egi-trustanchors]: https://github.com/indigo-iam/egi-trust-anchors-container/
[nginx-httpg-voms]: https://baltig.infn.it/cnafsd/ngx_http_voms_module
