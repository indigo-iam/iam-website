---
title: "Health checks"
linkTitle: "Health checks"
---

The IAM Login Service exposes a set of health endpoints that can be used to
monitor the status of the service.

Health endpoints expose a different set of information depending on the user
privileges; requests from the *Actuator* user will see more details, while
anonymous requests typically receive only a summary of the health status.

The Actuator role has been introduced in IAM starting with version 1.8.0,
in order to access the following resources
* `/actuator/health`
* `/actuator/info`

without being an IAM user and it is useful for deployment purposes.  
The Actuator user credentials can be configured with the environment variables
`IAM_ACTUATOR_USER_USERNAME` and `IAM_ACTUATOR_USER_PASSWORD`,
as explained in [Configuration](../../configuration/#basic-service-configuration).

The health endpoints return:
- HTTP status code 200 if everything is ok;
- HTTP status code 500 if any health check fails.

## `/actuator/health`

This is a general application health check endpoint which composes disk space
and database health checks.

Examples.

```console
$ curl -s https://iam.local.io/actuator/health | jq
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    },
    "diskSpace": {
      "status": "UP"
    },
    "ping": {
      "status": "UP"
    }
  }
}
```

Sending basic authentication, the endpoint returns a response with more details:

```console
$ curl -s -u $IAM_ACTUATOR_USER_USERNAME:$IAM_ACTUATOR_USER_PASSWORD https://iam.local.io/actuator/health | jq
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "H2",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 502468108288,
        "free": 320953249792,
        "threshold": 10485760,
        "exists": true
      }
    },
    "ping": {
      "status": "UP"
    }
  }
}

```

## `/actuator/health/mail`

This endpoint monitors the connection to the SMTP server configured for the
IAM Notification Service.  
In order to enable the SMTP server check, set the environment variable
`IAM_HEALTH_MAIL_PROBE_ENABLED` to true.

```console
$ curl -s https://iam.local.io/actuator/health/mail | jq
{
  "status": "UP"
}
```

With an authenticated request, the SMTP server details are returned:
```console
$ curl -u $IAM_ACTUATOR_USER_USERNAME:$IAM_ACTUATOR_USER_PASSWORD https://iam.local.io/actuator/health/mail | jq
{
  "status": "UP",
  "details": {
    "location": "smtp.local.io:25"
  }
}
```

## `/actuator/health/externalConnectivity`

This endpoint checks service connectivity to the Internet. By default, the
endpoint triggers a check on the connectivity to Google.  
In order to enable the external connectivity check, set the environment variable 
`IAM_HEALTH_EXTERNAL_CONNECTIVITY_PROBE_ENABLED` to true.

```console
$ curl -s https://iam.local.io/actuator/health/externalConnectivity | jq
{
  "status": "UP"
}
```

With an authenticated request, the external service URL is shown in the details.
```console
$ curl -s -u $IAM_ACTUATOR_USER_USERNAME:$IAM_ACTUATOR_USER_PASSWORD https://iam.local.io/actuator/health/externalConnectivity | jq
{
  "status": "UP",
  "details": {
    "endpoint": "https://www.google.it"
  }
}
```
