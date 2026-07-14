---
title: "Configuration"
linkTitle: "Configuration"
weight: 2
description: >
  IAM service configuration reference
---

Most IAM configurable aspects are configured via environment variables and
Spring profile directives.

## Spring profiles

Spring profiles are used to enable/disable group of IAM functionalities.
Currently the following profiles are defined:

| Profile name   | Active by default   | Description                                                                        |
| -------------- | ------------------- | ---------------------------------------------------------------------------------- |
| prod           | no                  | Sets up the environment for IAM in production                                      |
| h2-test        | yes                 | Enables h2 in-memory database, useful for development and testing                  |
| mysql-test     | no                  | Like h2-test, but used to develop against a MySQL database                         |
| oidc           | no                  | Enables OIDC authentication (e.g. Google)                                          |
| saml           | no                  | Enables SAML authentication                                                        |
| registration   | yes                 | Enables user registration and reset password functionalities                       |
| wlcg-scopes    | no                  | Enables WLCG token encoding                                                        |
| mfa            | no                  | Enables Multi-Factor Authentication (MFA) settings                                 |
| openid-federation | no                 | Allows IAM to act as an OpenID Provider and Relying Party in the OpenID Federation model                         |

Profiles are enabled by setting the `spring.profiles.active` Java system
property when starting the IAM service. This can be done, using the official
IAM docker image, by setting the IAM_JAVA_OPTS environment variable as follows:

```bash
IAM_JAVA_OPTS="-Dspring.profiles.active=prod,oidc,saml"
```

## Overriding default configuration templates

Fine-grained control over configuration can be obtained following the rules for
spring boot [externalized configuration][spring-boot-conf-rules].
This basically means defining one or more YAML files to override the default
configuration files embedded in the IAM Web Archive (war) package for the Spring profiles
activated for your instance.  The files must be placed in the IAM configuration
directory, which depends on how you deployed IAM: 

| Deployment type | Configuration directory |
|-----------------|-------------------------|
| Docker | `/indigo-iam/config/` |
| Package (RPM) | `/etc/indigo-iam/config` |

**IMPORTANT**: the default configuration should solve most use cases, override
default configuration **only if you know what you are doing**, and for those
scenarios not served by the default templates. 

## IAM login service

The value of the below environment variables are the default one configured into IAM and allows
to quickly run the application in development mode. They are not intended for a production environment.

### Basic service configuration 

```bash
# The IAM service will list for requests on this host.
# In a production environment it could be someting like 'iam.test.example'
IAM_HOST=localhost

# The IAM service webapp will bind on this port
IAM_PORT=8080

# The IAM web application base URL.
# We highly recommend to use https in production.
IAM_BASE_URL=http://${IAM_HOST}:8080

# The OpenID Connect issuer configured for this IAM instance.
# This must be equal to IAM_BASE_URL
IAM_ISSUER=http://${IAM_HOST}:8080

# The path to the JSON keystore that holds the keys IAM will use to sign and
# verify token signatures
IAM_KEY_STORE_LOCATION=classpath:keystore.jwks

# HTTP caching header setting public key lifetime (in seconds).
# The recommended lifetime according to the WLCG profile* is 6 hours
IAM_JWK_CACHE_LIFETIME=21600

# IAM will look for trust anchors in this directory. These trust anchors are
# needed for TLS operations where the IAM acts as a client (i.e., to
# authenticate to remote SAML Identity providers)
IAM_X509_TRUST_ANCHORS_DIR=/etc/grid-security/certificates

# How frequently (in seconds) should trust anchors be refreshed
IAM_X509_TRUST_ANCHORS_REFRESH=14400

# Use forwarded headers from reverse proxy. Set this to native when deploying the
# service behind a reverse proxy
IAM_FORWARD_HEADERS_STRATEGY=none

## Tomcat embedded container settings

# Enables the tomcat access log
IAM_TOMCAT_ACCESS_LOG_ENABLED=false

# Directory where the tomcat access log will be written (when enabled)
IAM_TOMCAT_ACCESS_LOG_DIRECTORY=/tmp

## Actuator endpoint settings

# Sets the username of the user allowed to have privileged access to actuator
# endpoints
IAM_ACTUATOR_USER_USERNAME=user

# Sets the password of the user allowed to have privileged access to actuator
# endpoints
IAM_ACTUATOR_USER_PASSWORD=secret

## Local resources configuration

# Enables the serving of resources from the local file system 
IAM_LOCAL_RESOURCES_ENABLE=false

# Sets the directory that contains the local resources that should be exposed
IAM_LOCAL_RESOURCES_LOCATION=file:/indigo-iam/local-resources
```
(*) More information [here][wlcg-profile].

### Organization configuration

```bash
# The name of the organization managed by this IAM instance
IAM_ORGANISATION_NAME=indigo-dc

# URL of logo image used in the IAM dashboard (by default the INDIGO-Datacloud
# project logo image is used)
IAM_LOGO_URL=resources/images/indigo-logo.png

# Size of the logo image (in pixels)
IAM_LOGO_DIMENSION=200

# Height of the logo image (in pixels)
IAM_LOGO_HEIGTH=150

# Width of the logo image (in pixels)
IAM_LOGO_WIDTH=200

# String displayed into the brower top bar when accessing the IAM dashboard
IAM_TOPBAR_TITLE="INDIGO IAM for ${IAM_ORGANISATION_NAME}"
```

### Database configuration

```bash
# Hostname of database server
IAM_DB_HOST=localhost

# Port of database server
IAM_DB_PORT=3306

# IAM database name
IAM_DB_NAME=iam

# The custom list of URL connection parameters. It must start with "?" character,
# if defined. The default value overrides the session time zone setting on the
# server to "UTC" and disables SSL usage
IAM_DB_URL_PARAMS=?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true

# Username for accessing the database
IAM_DB_USERNAME=iam

# Password for accessing the database
IAM_DB_PASSWORD=pwd

# The default maximum database connection pool size
IAM_DB_MAX_ACTIVE=50

# The minimum number of idle connections the pool attempts to maintain
IAM_DB_MIN_IDLE=8

# A configuration property used to validate the health of a database connection
# before it is handed out from the pool
IAM_DB_VALIDATION_QUERY=SELECT 1
```

### Access token contents configuration 

```bash
## Token content settings 

# Include authentication claims in issued access tokens
IAM_ACCESS_TOKEN_INCLUDE_AUTHN_INFO=false

# Includes the scope in issued access tokens
IAM_ACCESS_TOKEN_INCLUDE_SCOPE=false

# Includes the nbf claim in issued access tokens
IAM_ACCESS_TOKEN_INCLUDE_NBF=false

# Configures how long before the token's issue time it becomes valid
# The default value of 60 configures the token to be valid starting 60
# seconds before it is issued
IAM_ACCESS_TOKEN_NBF_OFFSET_SECONDS=60
```

### Persisting access tokens

```bash
# Set to 'true' if you want to store access tokens on the database
# (default behaviour). In this case, the token validation is performed
# by checking if the token is saved in the database.
# If set to 'false', the access tokens are not stored on the database,
# and the token validation is performed "offline"
IAM_ACCESS_TOKEN_STORE_ON_DATABASE=true
```

### Local authentication settings

It allows a user to log in with local credentials (username/password).  
For more information, see the [Local Authentication section][local-authn].

```bash
# Set to 'hidden' if you want to hide the local login form
IAM_LOCAL_AUTHN_LOGIN_PAGE_VISIBILITY=visible

# Enables local login form to all users.
# It can be restricted, changing the value to 'vo-admins' or 'none'
IAM_LOCAL_AUTHN_ENABLED_FOR=all
```

### Google authentication settings

```bash
# The Google OAuth client id
IAM_GOOGLE_CLIENT_ID=

# The OAuth client secret
IAM_GOOGLE_CLIENT_SECRET=
```

For more information and examples, see the [OpenID Connect
Authentication section]({{< ref "/docs/reference/configuration/external-authentication/oidc" >}}).

### SAML authentication settings

```bash
# The SAML entity ID for this IAM instance
IAM_SAML_ENTITY_ID=urn:iam:iam-devel

# Text shown in the SAML login button on the IAM login page
IAM_SAML_LOGIN_BUTTON_TEXT=Sign in with SAML

# Whether the WAYF discovery button is shown on the IAM login page
IAM_SAML_WAYF_LOGIN_BUTTON_VISIBLE=true

## SAML keystore settings

# The keystore holding certificates and keys used for SAML crypto
IAM_SAML_KEYSTORE=classpath:/saml/samlKeystore.jks

# The SAML keystore password
IAM_SAML_KEYSTORE_PASSWORD=password

# The identifier of the key that should be used to sign requests/assertions
IAM_SAML_KEY_ID=iam

# The password of the SAML key that will be used to sign requests/assertions
IAM_SAML_KEY_PASSWORD=password

## Metadata settings

# a URL pointing to the SAML federation or IdP metadata
IAM_SAML_IDP_METADATA=classpath:/saml/idp-metadata.xml

# Metadata refresh period (in seconds)
IAM_SAML_METADATA_LOOKUP_SERVICE_REFRESH_PERIOD_SEC=600

# Should signature validity checks be enforced on metadata?
IAM_SAML_METADATA_REQUIRE_VALID_SIGNATURE=false

# Trust only IdPs that have SIRTFI compliance
IAM_SAML_METADATA_REQUIRE_SIRTFI=false

# Comma-separated IDP entity ID whitelist. When empty
# all IdPs included in the metadata are whitelisted
IAM_SAML_IDP_ENTITY_ID_WHITELIST=

## Assertion validity settings

# Maxixum allowed assertion time (in seconds)
IAM_SAML_MAX_ASSERTION_TIME=3000

# Maximum authentication age (in seconds)
IAM_SAML_MAX_AUTHENTICATION_AGE=86400

## Other settings

# List of attribute aliases that are looked up in assertion to identify the 
# user authenticated with SAML
IAM_SAML_ID_RESOLVERS=eduPersonUniqueId,eduPersonTargetedId,eduPersonPrincipalName
```
{{% alert title="Warning" color="warning" %}}
In case you configure IAM such to support a SAML federation, the polling and
refreshing of the federation metadata (specified by the environment variable
`IAM_SAML_IDP_METADATA`) may encounter an out of memory issue.
Then, we reccomend to increase the heap space to 3 GB through the JVM options:
`IAM_JAVA_OPTS=-Xms3000m -Xmx3000m`.
{{% /alert %}}

For more information and examples, see the [SAML Authentication
section]({{< ref "/docs/reference/configuration/external-authentication/saml" >}}).

### Notification service settings

```bash
## SMTP mail server settings 

# SMTP server hostname 
IAM_MAIL_HOST=localhost

# SMTP server port 
IAM_MAIL_PORT=25

# SMTP server username
IAM_MAIL_USERNAME=

# SMTP server password
IAM_MAIL_PASSWORD=

## IAM notification settings

# Should the notification server be disabled?
# When set to true, notifications are not sent to the mail server (but
# printed to the logs)
IAM_NOTIFICATION_DISABLE=false

# The email address used as the sender in IAM email notifications
IAM_NOTIFICATION_FROM=indigo@localhost

# The email address used as the recipient in IAM email notifications
IAM_NOTIFICATION_ADMIN_ADDRESS=indigo-alerts@localhost

# Notification policy for account requests. The default value is notify-address
# meaning that notifications just arrive to the email address
# specified above. Set to notify-admins if you want to notify
# all IAM admins or to notify-address-and-admins to combine the two behaviors
IAM_NOTIFICATION_ADMIN_NOTIFICATION_POLICY=notify-address

# Should notifications be made when a profiles certificate information is updated?
# When set to true, notifications are made when a certifcate is linked or unlinked 
# from a profile. It follows the notifcation strategy of notification policy.
IAM_NOTIFICATION_CERTIFICATE=false

# Notification policy for group requests. Default value notifies both admins and
# group managers. Set to notify-gms if you want to notify only group managers
IAM_NOTIFICATION_GROUP_MANAGER_NOTIFICATION_POLICY=notify-gms-and-admins

# Time interval, in milliseconds, between two consecutive runs
# of IAM notification dispatch task 
IAM_NOTIFICATION_TASK_DELAY=30000

# Retention of delivered messages, in days
IAM_NOTIFICATION_CLEANUP_AGE=30
```

For more customization of the notification service settings, see the [IAM Notifications section]({{< ref "/docs/reference/configuration/notifications" >}}).

### Account linking settings

```bash
# Should account linking be disabled? When set to true users cannot
# link external accounts (Google, SAML) to their local IAM account
IAM_ACCOUNT_LINKING_DISABLE=false 
```

### Client registration

Those variables allow to configure how to register a new client and
the related default settings.

For more information see the [Client registration
section]({{< ref "/docs/reference/configuration/client-registration" >}}).

```bash
# Specifies who can register a client. Default is anyone, so also not registered
#  users. Other possible values are: REGISTERED_USERS and ADMINISTRATORS
IAM_CLIENT_REGISTRATION_ALLOW_FOR=ANYONE

# Set to false if you do not want to enable client registration (default is true)
IAM_CLIENT_REGISTRATION_ENABLE=true

# Set to true if you want only admin users to be able to create custom scopes
# (default is false)
IAM_CLIENT_ADMIN_ONLY_CUSTOM_SCOPES=false

# Set the default validity in seconds of an AT requested by any newly
# registered client. Default is 1 hour, but it can be changed per client
IAM_DEFAULT_ACCESS_TOKEN_VALIDITY_SECONDS=3600

# Set the default validity in seconds of a device code requested by any newly
# registered client. Default is 10 minutes, but it can be changed per client
IAM_DEFAULT_DEVICE_CODE_VALIDITY_SECONDS=600

# Set the default validity in seconds of an ID token requested by any newly
# registered client. Default is 10 minutes, but it can be changed per client
IAM_DEFAULT_ID_TOKEN_VALIDITY_SECONDS=600

# Set the default validity in seconds of an RT requested by any newly
# registered client. Default is 30 days, but it can be changed per client
IAM_DEFAULT_REFRESH_TOKEN_VALIDITY_SECONDS=2592000
```

### Client lifecycle

```bash
# Record the last date when each client was used to create or refresh a token
# This is used to determine which clients are considered active/inactive
IAM_CLIENT_TRACK_LAST_USED=false
```

### Privacy policy settings

```bash
# An URL pointing to a privacy policy document which applies
# to this IAM instance. When left blank, no privacy policy link 
# is displayed in the login page
IAM_PRIVACY_POLICY_URL=

# The text displayed in the login page for the privacy
# policy URL specified above
IAM_PRIVACY_POLICY_TEXT=Privacy policy
```
### Support Button Settings 

```bash 
# A URL that directs users to the page where they can open a support ticket 
# for this IAM instance. When left blank (which is the default), no support
# link is displayed in the login page
IAM_SUPPORT_URL=

# The text displayed in the login page for the support URL specified above,
# if enabled
IAM_SUPPORT_TEXT=Support
```

### mfa

```bash
# If set to 'false', users cannot enroll in MFA because
# the 'Enable MFA' button will NOT be shown.
IAM_TOTP_MFA_ENABLE_MFA_SETTINGS_BUTTON=true

# The current password used to encrypt and decrypt TOTP secrets.
# The default password MUST be changed from `define_me_please` to a preferred strong password.
IAM_TOTP_MFA_PASSWORD_TO_ENCRYPT_AND_DECRYPT=define_me_please

# The previous password, used ONLY to decrypt existing secrets during rotation.
# Leave this blank when no rotation is in progress.
IAM_TOTP_MFA_OLD_PASSWORD_TO_DECRYPT=

# If set to 'true', users will be forced to enroll in MFA.
IAM_MULTI_FACTOR_MANDATORY=false
```

### OpenID Federation

```bash
# URL of one or more trusted trust anchors. May be a comma-separated list
IAM_OIDFED_TRUST_ANCHORS=https://ta.example.com

# Validity duration (in seconds) of the generated Entity Configuration document
IAM_OIDFED_ENTITY_CONFIGURATION_EXPIRATION_SECONDS=86400

# One or more federation authorities that this entity trusts.
# May be a comma-separated list
IAM_OIDFED_ENTITY_CONFIGURATION_AUTHORITY_HINTS=https://ta.example.com,https://ia.example.com

# Human-readable name of the organization operating this federation entity.
# Filling this property is not mandatory
IAM_OIDFED_FEDERATION_ENTITY_ORGANIZATION_NAME=

# Contact info of administrators (e.g. email addresses).
# May be comma-separated. Not mandatory
IAM_OIDFED_FEDERATION_ENTITY_CONTACTS=

# URI of a logo representing the entity. Not mandatory
IAM_OIDFED_FEDERATION_ENTITY_LOGO_URI=
```

### Redis configuration

IAM supports storing HTTP session information and its in-memory cache
(for the well-known endpoint, scope matchers, etc.) in an external [redis][redis] server.

This can be useful when [deploying multiple replicas of the IAM
service](../../../docs/tasks/deployment/ha).

```bash
## Redis server settings

# Redis server hostname
IAM_SPRING_REDIS_HOST=localhost

# Redis server port
IAM_SPRING_REDIS_PORT=6397

# Redis server password.
# Leave it empty in case the server does not require any password
IAM_SPRING_REDIS_PASSWORD=secret

## Session settings

# Duration of an HTTP session
IAM_SESSION_TIMEOUT_SECS=1800

# Set to 'redis' in order to handle HTTP session
# with an external Redis service
IAM_SPRING_SESSION_STORE_TYPE=none

# If set to 'true' the status of the Redis service
# will appear in the IAM Health check endpoint
IAM_HEALTH_REDIS_PROBE_ENABLED=false

## Cache settings

# Enable the caching mechanism in IAM.
# When set to 'false', no-one kind of cache will be used.
# The default behavior is an in-memory cache
IAM_CACHE_ENABLED=true

# Refresh period for the external OIDC providers well-known
# endpoint cache (in seconds). Used during Proxied token introspection
# (disabled by default)
IAM_OIDC_DISCOVERY_CLEANUP_PERIOD_SECS=86400

# Allow to cache the IAM information into an external Redis service
IAM_CACHE_REDIS_ENABLED=false
```

## Test Client configuration

```bash
# Public identifier for client application
IAM_CLIENT_ID=client

# Client application's own password
IAM_CLIENT_SECRET=secret

# Default scopes allowed to the client application (optional)
IAM_CLIENT_SCOPES=openid profile email

# Use forwarded headers from reverse proxy. Set this to native when
# deploying the service behind a reverse proxy
IAM_CLIENT_FORWARD_HEADERS_STRATEGY=none
```

## VOMS AA

The environment variables for VOMS AA which tune the access to the database
are in common with IAM login service, even if they are sourced through a separated `mysql` profile
(i.e. they need to be defined for both IAM login service and VOMS AA when they run in different
container/machines). See [here](/docs/reference/configuration/#database-configuration) in order to set the proper variables.


Moreover, the VOMS AA-only environment variables are:

```bash
# Address the server binds to (network interface)
VOMS_AA_BINDING_ADDRESS=0.0.0.0

# Port the server listens on
VOMS_AA_PORT=8080

# Name of the Virtual Organization (VO)
VOMS_AA_VONAME=test

# Strategy for handling forwarded HTTP headers (e.g. from proxies).
# Set "native" if you're behind a proxy
VOMS_AA_FORWARD_HEADERS_STRATEGY=none

# Path to the TLS server certificate file
VOMS_AA_TLS_CERTIFICATE_PATH=/certs/hostcert.pem

# Path to the TLS private key file
VOMS_AA_TLS_PRIVATE_KEY_PATH=/certs/hostkey.pem

# Directory containing trusted CA certificates
VOMS_AA_TLS_TRUST_ANCHORS_DIR=/etc/grid-security/certificates

# Interval (in seconds) to reload trusted CA certificates
VOMS_AA_TLS_TRUST_ANCHORS_REFRESH_INTERVAL_SECS=14400

# Attribute name used for optional group membership
VOMS_AA_OPTIONAL_GROUP_LABEL=wlcg.optional-group

# Label name used for VOMS roles
VOMS_AA_VOMS_ROLE_LABEL=voms.role

# Enable legacy encoding format for FQAN attributes
VOMS_AA_USE_LEGACY_FQAN_ENCODING=false
```

{{% alert title="Warning" color="warning" %}}
A top level group equal to `VOMS_AA_VONAME` must be defined in IAM. The attributes appearing in the VOMS proxy include all the sub-groups of the parent group equal to the VO name. Top level groups different from the VO name may still be used for group-based authorization with JWTs, but will not appear in the proxy.
{{% /alert %}}

[spring-boot-conf-rules]: https://docs.spring.io/spring-boot/docs/1.3.8.RELEASE/reference/html/boot-features-external-config.html
[redis]: https://redis.io/
[wlcg-profile]: https://github.com/WLCG-AuthZ-WG/common-jwt-profile/blob/master/profile.md#token-validation
[local-authn]: local_authn/
