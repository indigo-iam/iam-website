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
| prod           | no                  | This is the profile you should enable when using IAM                               |
| h2-test        | yes                 | Enables h2 in-memory database, useful for development and testing                  |
| mysql-test     | no                  | Like h2-test, but used to develop against a MySQL database                         |
| oidc           | no                  | Enables Google authentication                                                      |
| saml           | no                  | Enables SAML authentication                                                        |
| registration   | yes                 | Enables user registration and reset password functionalities                       |
| wlcg-scopes    | no                  | This is the profile you should enable when you want to support WLCG token encoding |

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

## Basic service configuration 

```bash
# The IAM service will list for requests on this host
IAM_HOST=localhost

# The IAM service webapp will bind on this port
IAM_PORT=8080

# The IAM web application base URL
IAM_BASE_URL=http://${IAM_HOST}:8080

# The OpenID Connect issuer configured for this IAM instance.
# This must be equal to IAM_BASE_URL
IAM_ISSUER=http://${IAM_HOST}:8080

# The path to the JSON keystore that holds the keys IAM will use to sign and
# verify token signatures
IAM_KEY_STORE_LOCATION=

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

## Organization configuration

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

## Access token contents configuration 

```bash
## Token content settings 

# Include authentication claims in issued access tokens
IAM_ACCESS_TOKEN_INCLUDE_AUTHN_INFO=false

# Includes the scope in issued access tokens
IAM_ACCESS_TOKEN_INCLUDE_SCOPE=false

# Includes the nbf claim in issued access tokens
IAM_ACCESS_TOKEN_INCLUDE_NBF=false
```

## Database configuration

```bash
# The host where the MariaDB/MySQL daemon is running
IAM_DB_HOST=

# The database port
IAM_DB_PORT=3306

# The database name
IAM_DB_NAME=iam

# The database username
IAM_DB_USERNAME=iam

# The database password 
IAM_DB_PASSWORD=pwd

## Database connection pool options

# Maximum number of active connections to the database
IAM_DB_MAX_ACTIVE=50

# Maximum number of idle connections in the pool 
IAM_DB_MAX_IDLE=5

# Initial size of the database connection pool
IAM_DB_INITIAL_SIZE=8

# Should idle connections in the pool be tested?
IAM_DB_TEST_WHILE_IDLE=true

# Should connections in the pool be tested when borrowed?
IAM_DB_TEST_ON_BORROW=true

# Which SQL query should be used to test connections?
IAM_DB_VALIDATION_QUERY=SELECT 1

# Time between database connection pool eviction runs (in msec)
IAM_DB_TIME_BETWEEN_EVICTION_RUNS_MILLIS=5000

# The minimum amount of time a connection may be idle in the pool
# before it is considered for eviction (in msec)
IAM_DB_MIN_EVICTABLE_IDLE_TIME_MILLIS=60000
```

## Test Client configuration

```bash
# Public identifier for client application
IAM_CLIENT_ID=client

# Client application's own password
IAM_CLIENT_SECRET=secret

# Default scopes allowed to the client application (optional)
IAM_CLIENT_SCOPES=openid profile email

# Use forwarded headers from reverse proxy. Set this to native when deploying the
# service behind a reverse proxy
IAM_CLIENT_FORWARD_HEADERS_STRATEGY=none
```

## Redis configuration

Starting with version 1.8.0, IAM supports storing HTTP session
information in an external [redis][redis] server.

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

# Duration of an HTTP session
IAM_SESSION_TIMEOUT_SECS=1800

# Set to 'redis' in order to handle HTTP session
# with an external Redis service
IAM_SPRING_SESSION_STORE_TYPE=none

# If set to 'true' the status of the Redis service
# will appear in the IAM Health check endpoint
IAM_HEALTH_REDIS_PROBE_ENABLED=false
```

## Local authentication settings

It allows a user to log in with local credentials (username/password).  
For more information, see the [Local Authentication section][local-authn].

```bash
# Set to 'hidden' if you want to hide the local login form
IAM_LOCAL_AUTHN_LOGIN_PAGE_VISIBILITY=visible
# Enables local login form to all users.
# It can be restricted, changing the value to 'vo-admins' or 'none'
IAM_LOCAL_AUTHN_ENABLED_FOR=all
```

## Google authentication settings

```bash
# The Google OAuth client id
IAM_GOOGLE_CLIENT_ID=

# The OAuth client secret
IAM_GOOGLE_CLIENT_SECRET=
```

For more information and examples, see the [OpenID Connect
Authentication section]({{< ref "/docs/reference/configuration/external-authentication/oidc" >}}).

## SAML authentication settings

```bash
# The SAML entity ID for this IAM instance
IAM_SAML_ENTITY_ID=

# Text shown in the SAML login button on the IAM login page
IAM_SAML_LOGIN_BUTTON_TEXT=Sign in with SAML

## SAML keystore settings

# The keystore holding certificates and keys used for SAML crypto
IAM_SAML_KEYSTORE= 

# The SAML keystore password
IAM_SAML_KEYSTORE_PASSWORD=

# The identifier of the key that should be used to sign requests/assertions
IAM_SAML_KEY_ID=

# The password of the SAML key that will be used to sign requests/assertions
IAM_SAML_KEY_PASSWORD=

## Metadata settings

# a URL pointing to the SAML federation or IdP metadata
IAM_SAML_IDP_METADATA=

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

For more information and examples, see the [SAML Authentication
section]({{< ref "/docs/reference/configuration/external-authentication/saml" >}}).

## Notification service settings

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

# Notification policy for group requests. Default value notifies both
# admins and group managers. Set to notify-gms if you want to notify only group managers
IAM_NOTIFICATION_GROUP_MANAGER_NOTIFICATION_POLICY=notify-gms-and-admins

# Time interval, in milliseconds, between two consecutive runs of IAM notification 
# dispatch task 
IAM_NOTIFICATION_TASK_DELAY=30000

# Retention of delivered messages, in days
IAM_NOTIFICATION_CLEANUP_AGE=30
```
## Account linking settings

```bash
# Should account linking be disabled? When set to true users cannot
# link external accounts (Google, SAML) to their local IAM account
IAM_ACCOUNT_LINKING_DISABLE=false 
```
## Privacy policy settings

```bash
# An URL pointing to a privacy policy document which applies
# to this IAM instance. When left blank, no privacy policy link 
# is displayed in the login page
IAM_PRIVACY_POLICY_URL=

# The text displayed in the login page for the privacy policy URL specified
# above
IAM_PRIVACY_POLICY_TEXT=Privacy policy
```

[spring-boot-conf-rules]: https://docs.spring.io/spring-boot/docs/1.3.8.RELEASE/reference/html/boot-features-external-config.html
[redis]: https://redis.io/
[wlcg-profile]: https://github.com/WLCG-AuthZ-WG/common-jwt-profile/blob/master/profile.md#token-validation
[local-authn]: local_authn/
